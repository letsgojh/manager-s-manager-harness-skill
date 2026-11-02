# Manager's Manager AI 하네스 (Claude Code 플러그인)

[Manager's Manager](https://github.com/kakaotechcampus-4/ktc4-kyungpook-3) 저장소의 `ai/` 파트 전용 Claude Code 하네스다. 서비스 저장소와 목적이 달라 따로 둔다. 지금은 그 저장소 구조(`ai/.venv`, `ai/CLAUDE.md`, develop 대상 PR)를 전제로 한다. 일반화는 나중에 한다.

## 설치

Claude Code 에서:

```
/plugin marketplace add letsgojh/manager-s-manager-harness-skill
/plugin install mm-harness@mm-harness-marketplace
```

[Superpowers](https://github.com/obra/superpowers) 플러그인(`superpowers@claude-plugins-official`)에 의존한다. 함께 설치되지 않으면 먼저 `/plugin install superpowers@claude-plugins-official` 를 실행한다. 설치 후 Claude Code 를 다시 시작하고, 서비스 저장소 루트에서 실행한다.

업데이트: `/plugin marketplace update mm-harness-marketplace` 후 `/plugin update mm-harness@mm-harness-marketplace`.

## 명령

| 명령 | 하는 일 |
|---|---|
| `/mm-harness:ai-feature <#이슈번호 \| 기능 설명>` | 설계 → 테스트 먼저 구현 → pytest → 셀프 리뷰 → develop 대상 PR 생성. 이슈로 부르면 설계 확정을 이슈 댓글로 남기고 PR 에 `Closes` 를 단다. BE·FE 에 전달할 것이 있을 때만 `ai→be`·`ai→fe` 이슈 초안을 만들어 확인받고 올린다. 머지는 사람이 한다 |
| `/mm-harness:ai-pr-reply [PR번호]` | PR 리뷰를 모아 지적마다 판정. 맞는 지적은 고쳐서 push 후 답글, 반박·질문 답글은 사용자 승인 후 단다 |
| `/mm-harness:ai-issue-reply <이슈번호[,번호...]>` | 다른 파트에 넘긴 이슈(`ai→be`·`ai→fe`·`contract`)의 답을 항목마다 대조하고, 수정안·질문은 사용자와 정한 뒤 승인받은 답글을 단다. 합의되면 `합의됨` 라벨, 계약 문서 PR, 후속 AI 요청 이슈 정리. 코드는 고치지 않는다. 사용자가 부를 때만 동작 |
| `/mm-harness:ai-review-pr [PR번호]` | 팀원 PR 을 AI 관점에서 리뷰한다. 번호가 없으면 열린 PR 을 훑어 리뷰할 PR 을 추천하고 사용자가 고른다. `ai-reviewer` 로 리뷰한 뒤 코멘트 초안을 승인받아 `COMMENT` 리뷰로 올린다(승인·변경 요청 상태는 정하지 않음). 사용자가 부를 때만 동작 |
| `/mm-harness:grill-me` | 설계·계획만 따로 질문 라운드로 다듬는다 |
| `/mm-harness:teach <대상>` | 변경·PR·설계·서브시스템을 쉽게 설명한다. how·why 절차를 돌려 하나의 설명으로 엮는다. 사용자가 부를 때만 동작 |
| `/mm-harness:how <질문>` | "X 는 어떻게 동작하나". 코드를 탐색해 구조와 실행 흐름을 설명한다. 사용자가 부를 때만 동작 |
| `/mm-harness:why <질문>` | "왜 이렇게 했나". 결정 기록·계약·이슈·PR·git 이력을 조사해 근거와 확신 수준을 붙여 답한다. 사용자가 부를 때만 동작 |

## 구성

| 구성 | 파일 | 역할 |
|---|---|---|
| 에이전트 `ai-designer` | `agents/ai-designer.md` | opus, 읽기 전용. 설계 초안·크기 판정(small/large)·사람이 정할 질문 |
| 에이전트 `ai-implementer` | `agents/ai-implementer.md` | sonnet. 태스크 하나를 TDD 로 구현·커밋. `superpowers:test-driven-development` 를 미리 로드 |
| 에이전트 `ai-reviewer` | `agents/ai-reviewer.md` | sonnet, 읽기 전용. diff 리뷰 |
| 스킬 `ai-feature` | `skills/ai-feature/` | 기능 하네스. small 은 메인 세션에서, large 는 grilling 으로 설계 확정 후 `superpowers:subagent-driven-development` |
| 스킬 `ai-pr-reply` | `skills/ai-pr-reply/` | PR 리뷰 답변. `superpowers:receiving-code-review` 원칙 |
| 스킬 `ai-issue-reply` | `skills/ai-issue-reply/` | 다른 파트 이슈 답변 처리. 계약 결정은 `grilling` 으로 사람이 정한다. `disable-model-invocation` |
| 스킬 `ai-review-pr` | `skills/ai-review-pr/` | 팀원 PR 리뷰. 리뷰할 PR 추천 → 사용자 선택 → `ai-reviewer` → 초안 승인 → 리뷰 하나로 올림 |
| 스킬 `grilling`, `grill-me` | `skills/grilling/`, `skills/grill-me/` | 설계를 사용자와 질문 라운드로 정한다. 질문마다 배경·선택지별 결과·추천 이유를 쓰고, 라운드마다 이해한 결정을 확인한다. 아래 출처 참고 |
| 스킬 `how`, `why`, `teach` | `skills/how/`, `skills/why/`, `skills/teach/` | 사용자가 모르는 것을 물을 때 쓰는 설명 스킬. `disable-model-invocation` 이라 파이프라인 중에 자동으로 끼어들지 않는다. 아래 출처 참고 |
| Stop 훅 | `hooks/ai-tests-gate.sh` | `ai/*.py` 가 바뀐 채 응답을 끝내면 `ai/.venv` 로 pytest. 실패하면 끝내지 못하게 막고, 같은 상태로 통과했으면 건너뛰고, 연속 3번 막으면 연다. `ai/.venv` 가 없는 저장소에서는 아무것도 하지 않는다 |

## 서비스 저장소에 필요한 것

- 이슈 템플릿 `.github/ISSUE_TEMPLATE/ai-feature.md`: `ai-feature` 요청 양식 (목표, 먼저 읽을 것, 범위, 지킬 것, 정할 것, 완료 조건, 선행 이슈)
- 라벨 `ai→be`, `ai→fe`, `contract`, `합의됨`: 다른 파트 전달과 선행 계약 확인에 쓴다
- 워크플로 `.github/workflows/ai-handoff-agreed.yml`: 넘긴 이슈가 합의되면 기다리던 AI 요청 이슈를 팀 디스코드로 알린다. 하네스는 사용자 컴퓨터 세션에서 돌아 웹훅이 직접 깨울 수 없으므로, 알림을 보고 다시 시작하거나 세션을 켜 두고 `/loop` 로 감시한다

## 플러그인에 담지 못하는 설정

플러그인은 세션 기본 모델이나 권한 설정을 바꾸지 못한다. 필요하면 각자 설정에 둔다.

- 기본 모델: 에이전트 모델은 고정돼 있다. 메인 세션은 `/model sonnet` 을 권장한다 (opus 는 설계 에이전트만)
- `.env` 읽기·편집 차단은 서비스 저장소의 `.claude/settings.json` 에 있다

## 출처

- `skills/grilling`, `skills/grill-me`: [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, Copyright (c) 2026 Matt Pocock). 각 디렉터리의 `LICENSE` 참고. `grill-me` 는 플러그인 이름공간에 맞춰 호출할 스킬 이름만 바꿨다
- `skills/how`, `skills/why`, `skills/teach`: [michael-denyer/pstack-claude](https://github.com/michael-denyer/pstack-claude) (MIT, Copyright (c) 2026 Lauren Tan, Michael Denyer, 원작 pstack by Lauren Tan). 각 디렉터리의 `LICENSE` 참고. Claude Code 전용으로 줄이고 한국어로 옮기고 이 저장소의 근거 출처를 더했다
