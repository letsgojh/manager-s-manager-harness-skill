---
name: ai-feature
description: ai/ 파트 기능 하나를 받아 설계 → 테스트 먼저 구현 → pytest → 셀프 리뷰 → develop 대상 PR 생성까지 역할별 에이전트로 돌리고, PR 을 올리면 끝낸다. 사용자가 AI 파트(capture·stt·extract·judge·draft·shared) 기능 구현이나 버그 수정을 맡기거나 /ai-feature 로 부를 때 쓴다. 다른 파트 작업, 문서만 고치는 작업, PR 리뷰 답변(ai-pr-reply)에는 쓰지 않는다.
argument-hint: "[#이슈번호 | 이슈 URL | 기능 설명 | 계획 경로]"
---

# AI 파트 기능 하네스

요청: $ARGUMENTS

끝은 **develop 대상 PR 생성**이다. 머지는 사람이 PR 을 확인한 뒤 한다. 규칙은 `ai/CLAUDE.md` 와 `docs/agent-rules.md` 를 따른다.
이 플러그인의 에이전트와 스킬은 `mm-harness:` 이름공간 아래에 있다. 에이전트를 부를 때는 `mm-harness:ai-designer` 처럼 이름공간을 붙인다.
Superpowers 스킬을 쓰되, 이 문서가 정한 부분은 Superpowers 기본값보다 우선한다.
설계는 1단계(`ai-designer` 초안 + `grilling` 으로 사용자와 확정)가 맡으므로 `superpowers:brainstorming` 은 부르지 않는다. `using-superpowers` 가 brainstorming 을 먼저 하라고 해도 이 문서의 1단계가 그 자리다.

역할과 모델:

| 역할 | 누가 | 모델 | 쓰는 스킬 |
|---|---|---|---|
| 설계 초안 | `ai-designer` | opus | |
| 설계 확정 (사용자와) | 메인 세션 | 세션 모델 | `grilling` |
| 계획·태스크 실행 (large) | 메인 세션 | 세션 모델 | `superpowers:writing-plans`, `superpowers:subagent-driven-development` |
| 태스크 구현 | `ai-implementer` (small 은 메인 세션) | sonnet | `superpowers:test-driven-development` (미리 로드) |
| 리뷰 | `ai-reviewer` | sonnet | |

에이전트를 부를 때는 브리프에 태스크 설명과 파일 경로만 넣는다. 파일 내용을 붙여 넣지 않는다.

## 0. 준비

- **요청이 이슈면 이슈를 요청으로 쓴다.** `$ARGUMENTS` 가 `#123`, `123`, 이슈 URL 이면 `gh issue view <번호> --json number,title,body,labels,comments` 로 읽고 본문을 요청으로 쓴다. 이 번호를 "요청 이슈"로 기억한다(설계 댓글, PR 의 `Closes`). 이슈가 아니면 지금처럼 `$ARGUMENTS` 를 요청으로 쓰고 요청 이슈는 없다.
- **선행 계약 확인.** 요청 이슈 본문의 "선행 이슈"에 적힌 이슈마다 `gh issue view <번호> --json state,labels` 로 본다. 닫혔거나 `합의됨` 라벨이 있으면 통과다. 하나라도 아니면 설계까지는 하되, 구현으로 넘어가기 전에 그 사실을 알리고 묻는다.
- `git status` 를 본다. 이번 기능과 무관한 커밋 안 된 변경이 있으면 멈추고 묻는다.
- `git fetch origin develop` 후 `origin/develop` 에서 `feature/<짧은-영문-slug>` 브랜치를 만든다. 이미 이 기능의 feature 브랜치에 있으면 그대로 쓴다.
- **git 워크트리를 만들지 않는다.** `ai/.venv` 가 git 밖에 있어서 워크트리에서는 pytest 와 Stop 훅 게이트가 돌지 않는다. 지금 체크아웃의 feature 브랜치가 격리 작업 공간이다 (`using-git-worktrees` 를 쓰지 않는다).

## 1. 설계

1. `ai-designer` 에게 요청과 관련 경로를 넘겨 설계 초안을 받는다. 코드 읽기는 여기서 끝낸다.
2. 초안의 **크기 판정**으로 경로를 정한다. 판정이 애매하면 `large`.
3. 사용자와 설계를 정한다. 이 하네스에서 사람을 기다리는 곳은 여기와 4단계의 다른 파트 전달(전달할 것이 있을 때만) 둘이다.
   - `large`, 또는 `small` 인데 초안에 "사람이 정할 질문"이 있으면 `mm-harness:grilling` 스킬로 사용자와 같이 정한다.
     초안을 먼저 짧게 보여주고(목표·추천 접근안·변경 범위·계약 영향), 초안의 질문과 기본값을 첫 라운드의 출발점으로 쓴다.
     사실 확인이 더 필요하면 `Explore` 에이전트에 맡긴다. 초안에 이미 있는 사실은 다시 찾지 않는다.
     `grilling` 이 끝나고 사용자가 합의를 확인하면 구현으로 넘어간다.
   - `small` 이고 질문이 "없음"이면 묻지 않고 진행한다. 설계는 PR 본문에 남아 사람이 확인한다.
4. `large` 는 합의된 설계(초안 + grilling 에서 정한 것)를 `.superpowers/specs/YYYY-MM-DD-<slug>.md` 에 저장한다. git 에 올리지 않는 작업 파일이다.
5. 요청 이슈가 있고 grilling 을 했으면, 합의를 확인받을 때 "이 요약을 이슈 댓글로 남긴다"고 같이 말하고, 확인되면 `gh issue comment <번호> --body-file <파일>` 로 남긴다. 댓글은 `## 설계 확정 (AI 하네스)` 제목에 정한 것(질문마다 결론 한 줄), 크기, 계약 영향, 다른 파트 전달 후보를 적는다. `.superpowers/` 는 git 밖이라 팀원이 볼 수 있는 기록은 이 댓글이다.

다음 경우에는 크기와 상관없이 멈추고 묻는다: `shared/schemas.py` 나 BE API 계약 변경, 비용이 드는 실제 API 호출(LLM·STT·임베딩)이 필요할 때.

## 2. 구현

### small

`superpowers:test-driven-development` 대로 메인 세션에서 직접 한다. 서브에이전트를 띄우지 않는다.
테스트 먼저 → 예상한 이유로 실패 확인 → 최소 구현 → 전체 pytest (`ai/` 안에서 `.venv/bin/python -m pytest -q`) → 커밋.

### large

1. `superpowers:writing-plans` 로 계획을 쓴다. 저장 위치는 `.superpowers/plans/YYYY-MM-DD-<slug>.md` 다. 마일스톤 `plan.md` 나 전체 계획 파일은 쓰지 않는다. 실행 방식은 묻지 않고 subagent-driven 으로 간다.
2. `superpowers:subagent-driven-development` 로 실행한다. 단:
   - 구현자는 `ai-implementer`, 태스크 리뷰어와 최종 리뷰어는 `ai-reviewer` 로 부른다.
   - 최종 리뷰는 `shared/schemas.py` 나 여러 모듈에 걸친 변경일 때만 `model: opus` 로 올린다. 나머지는 에이전트 기본 모델을 쓴다.
   - 수정 라운드는 태스크당 최대 2회. 남은 지적은 ledger 에 판정과 함께 남기고 PR 본문으로 옮긴다.
   - 마지막의 `finishing-a-development-branch` 메뉴는 띄우지 않는다. 이 문서의 4단계로 간다.

응답을 끝낼 때마다 Stop 훅이 `ai/` pytest 를 돌리고 실패하면 끝내지 못하게 막는다. 막히면 출력된 실패를 고친다. 테스트를 지우거나 단언을 약하게 만들어 통과시키지 않는다.

## 3. 셀프 리뷰

- `small`: `ai-reviewer` 를 한 번 부른다. 범위는 `origin/develop...HEAD`. `blocker`·`should-fix` 를 고치고 전체 pytest 를 다시 돌린다. 재리뷰는 하지 않는다.
- `large`: subagent-driven-development 의 최종 리뷰가 셀프 리뷰다. 따로 더 돌리지 않는다.
- 고치지 않은 지적은 이유와 함께 PR 본문에 남긴다. 사람이 마지막에 본다.

## 4. 기록

- 여러 선택지 중 하나를 고른 결정이 있었으면 `decision-record` 스킬로 `ai/docs/basis_for_decision/ai/` 에 남긴다. 단순 구현은 남기지 않는다.
- `shared/schemas.py` 나 BE 호출이 바뀌었으면 `contract-update` 스킬을 쓴다.
- 대상 마일스톤이 명확할 때만 `milestone-result` 스킬을 쓴다.
- 기록도 커밋에 포함한다 (`docs(ai): ...`).

### 다른 파트 전달 (해당할 때만)

**전달할 내용이 있을 때만** 한다. 기능마다 습관적으로 이슈를 만들지 않는다. 설계 초안의 "다른 파트 전달 후보"와 구현·셀프 리뷰에서 나온 것을 모아 아래 셋 중 하나에 해당하는지 판정한다.

- **계약 변경**: `shared/schemas.py` 나 BE 호출이 바뀌어 BE 가 받는 쪽을 맞춰야 한다.
- **다른 파트 몫의 일**: 이 기능이 실제로 돌려면 BE·FE 가 해야 할 일이 있다 (화면, API, 상태, 형식 등).
- **다른 파트의 결정 필요**: AI 만으로 정할 수 없어 BE·FE 의 합의나 확인이 필요하다.

`ai/` 안에서 끝나는 변경(프롬프트, 내부 리팩터링, AI 버그 수정)은 해당하지 않는다. 해당이 없으면 PR 본문의 "다른 파트 전달"에 "없음"을 쓰고 넘어간다.

해당하면:
1. 받는 파트마다 이슈 초안을 만든다. 라벨은 `ai→be` 또는 `ai→fe`, 계약 변경이면 `contract` 도 붙인다. 본문 양식:
   ```markdown
   ## 배경
   <AI 쪽에서 무엇을 했고 왜 이 전달이 생겼는지 2~3줄. 관련 AI PR·이슈 링크>

   ## 요청
   - [ ] <받는 파트가 할 일 또는 정할 것. 하나씩 확인 가능한 문장>

   ## 명세 (있으면)
   <필드·API 모양. 합의 전이면 "제안"이라고 적는다>

   ## 확인할 것
   - [ ] <받는 파트가 답해야 할 질문>

   ## 근거
   - <코드 `파일:줄`, 계약 문서, 결정 기록 경로>
   ```
2. 초안 전체를 사용자에게 보여주고 확인받는다. 이슈는 팀원에게 보이므로 확인 없이 만들지 않는다. 고칠 것이 있으면 고쳐서 다시 보인다.
3. 확인되면 `gh issue create --label ... --body-file ...` 로 만든다. 요청 이슈가 있으면 본문 "근거"에 링크한다.
4. 만든 이슈 번호를 PR 본문 "다른 파트 전달"에 적는다. 계약 변경 이슈가 합의 전이면 PR 의 "사람이 확인할 것"에도 적는다.

## 5. PR 생성 (여기서 끝)

1. 전체 pytest 를 한 번 더 돌려 통과를 확인한다.
2. `git push -u origin <브랜치>` 후 `gh pr create --base develop` 로 PR 을 만든다. 제목은 대표 커밋 형식 `type(scope): 한국어 요약`.
3. PR 본문 (한국어):

```markdown
## 요약
<무엇을 왜 바꿨는지 2~3줄>

## 설계
<완료 조건, 고른 접근안과 이유, 계약 영향>

## 변경
- <파일 또는 모듈>: <무엇을 바꿨는지>

## 검증
- `<실제로 돌린 명령>`: <통과·실패 수>
- 돌리지 못한 것: <없으면 "없음">

## 셀프 리뷰
- 고친 것: ...
- 남긴 것과 이유: ...

## 사람이 확인할 것
- <에이전트가 혼자 판정한 것(ruling), 위험, 확신 없는 부분>

## 기록
- <결정 기록·계약 문서 경로, 없으면 "없음">

## 다른 파트 전달
- <만든 이슈 번호와 한 줄 요약, 없으면 "없음">

<요청 이슈가 있으면: Closes #<번호>>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

4. 사용자에게 PR 링크, "사람이 확인할 것", 만든 다른 파트 이슈만 짧게 보고하고 끝낸다. 머지, 리뷰어 지정, 리뷰 답변은 하지 않는다. PR 에 리뷰가 달리면 `/mm-harness:ai-pr-reply <PR번호>` 로 답한다.
