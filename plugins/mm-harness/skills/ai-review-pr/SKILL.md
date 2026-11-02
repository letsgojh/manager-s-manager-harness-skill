---
name: ai-review-pr
description: 팀원이 올린 PR 을 AI 파트 관점에서 리뷰한다. PR 번호가 없으면 열린 PR 들을 훑어 리뷰할 만한 PR 을 골라 사용자에게 추천하고, 사용자가 고른 PR 을 ai-reviewer 로 리뷰한 뒤 코멘트 초안을 보여 주고 승인받은 것만 PR 에 단다. 사용자가 /mm-harness:ai-review-pr 로 부를 때만 쓴다. 내 PR 에 달린 리뷰에 답하는 일(ai-pr-reply)이나 내 작업의 셀프 리뷰(ai-feature 3단계)에는 쓰지 않는다.
argument-hint: "[PR번호[,번호...], 비우면 리뷰할 PR 을 골라 추천]"
disable-model-invocation: true
---

# 팀원 PR 리뷰

대상: $ARGUMENTS (비어 있으면 1단계에서 고른다)

이 플러그인의 에이전트와 스킬은 `mm-harness:` 이름공간 아래에 있다. 리뷰는 `mm-harness:ai-reviewer` 에이전트에 맡긴다.
코드를 고치지 않는다. PR 브랜치를 체크아웃하지 않고 `gh pr diff`, `gh pr view`, `git show origin/<브랜치>:<경로>` 로 읽는다(작업 중인 브랜치를 건드리지 않기 위해서다).
PR 에 무엇이든 올리는 일(코멘트, 리뷰 상태)은 동료에게 보이므로 **사용자 승인 뒤에만** 한다.

## 1. 리뷰할 PR 고르기 (번호를 받지 않았을 때)

1. 나(`gh api user -q .login`)를 확인하고 열린 PR 을 모은다:
   ```bash
   gh pr list --state open --limit 50 --json number,title,author,headRefName,baseRefName,isDraft,createdAt,updatedAt,reviewRequests,reviews,files,labels
   ```
2. 뺀다: 내가 쓴 PR, draft, 내가 이미 리뷰했고 그 뒤로 새 커밋이 없는 PR(`gh pr view <N> --json reviews,commits` 로 내 마지막 리뷰 시각과 마지막 커밋 시각 비교).
3. 남은 PR 마다 **AI 관련도**를 매긴다. 근거를 한 줄로 남긴다.
   - **높음**: `ai/` 를 바꾸는 PR. 또는 AI 가 부르는 BE API(`backend/app/api/` 의 회의·발화·추출·업로드·유사 검색·연동 경로)나 `backend/docs/contracts/backend-ai-contract.md` 를 바꾸는 PR. 또는 본문·연결 이슈가 `ai→be`·`ai→fe`·`contract` 라벨 이슈나 AI 요청 이슈를 가리키는 PR.
   - **중간**: AI 결과를 보여 주는 FE 화면(추출 항목, 근거, 화자 표시, 회의 상태)이나 `frontend/docs/contracts/` 를 바꾸는 PR.
   - **낮음**: 그 밖의 것.
4. 우선순위를 정한다: 나에게 리뷰 요청이 온 PR > AI 관련도 높음 > 내 작업을 막고 있는 PR(내 열린 이슈의 "선행 이슈"가 가리키는 BE 이슈를 닫는 PR) > 오래 기다린 PR.
5. 사용자에게 표로 추천하고 **고르게 한다**. 기본으로 추천 상위 1~3개에 표시해 둔다.

   | 추천 | PR | 작성자 | 파트 | AI 관련도와 근거 | 기다린 시간 | 리뷰 관점 |
   |---|---|---|---|---|---|---|
   | ✅ | #N 제목 | @login | BE | 높음: 업로드 API 변경, #255 합의 대상 | 2일 | 합의와 다른 점, AI 재시도 시 동작 |

   리뷰 관점은 이 PR 에서 무엇을 중심으로 볼지 한 줄(예: "#255 합의와 다른 점", "`ai/` 규칙 위반과 테스트"). 관련도 낮음 PR 은 "AI 관점에서 볼 것이 적다"고 적고 기본 추천에서 뺀다.
6. 사용자가 고르면 다음 단계로 간다. 아무것도 고르지 않으면 끝낸다.

## 2. 리뷰 준비 (PR 마다)

- `gh pr view <N> --json title,body,author,headRefName,baseRefName,files,commits,closingIssuesReferences,url` 로 읽는다. 본문의 "봐 주셨으면 하는 곳"이 있으면 리뷰 관점에 더한다.
- 리뷰 범위: `gh pr diff <N>` (쌓인 PR 이면 base 브랜치 기준 diff 가 이미 나온다).
- 기준 문서를 PR 의 파트에 맞춰 고른다:
  - AI PR: `ai/CLAUDE.md`, 관련 결정 기록(`ai/docs/basis_for_decision/`), 연결 이슈의 설계 확정 댓글.
  - BE PR: `backend/docs/contracts/backend-ai-contract.md`(PR 이 고친 버전과 develop 버전), 관련 합의 이슈(본문·연결 이슈에서 찾은 `ai→be`·`contract` 이슈의 합의 댓글), AI 쪽 호출 코드(`ai/capture/handoff.py`, `ai/upload/` 등).
  - FE PR: `frontend/docs/contracts/frontend-api-contract.md`, 연결 이슈.

## 3. 리뷰 (PR 마다)

`mm-harness:ai-reviewer` 에 브리프로 넘긴다. 브리프에는 PR 번호, diff 를 얻는 명령(`gh pr diff <N>`), 기준 문서 경로, 리뷰 관점만 넣는다. 파일 내용을 붙여 넣지 않는다. 여러 PR 을 골랐으면 PR 마다 하나씩 동시에 띄운다.
- AI PR: 에이전트 기본 관점 그대로(정확성, `ai/` 규칙, 테스트, 단순화).
- BE·FE PR: "이 파트의 내부 규칙은 판단하지 말고, (1) 합의·계약과 다른 점, (2) AI 가 이 API 를 부를 때 깨지는 경우(재시도·멱등·오류 코드·필드 이름·시간대), (3) 명백한 버그와 테스트 누락만 본다"고 적는다.
- 변경이 크거나 계약에 걸치면 `model: opus` 로 부른다.

받은 지적은 메인 세션에서 다시 확인한다. diff 의 해당 줄을 직접 열어 근거가 맞는지 보고, 틀린 지적은 버린다. 확신이 없으면 "질문" 형태로 바꾼다.

## 4. 코멘트 초안과 승인

PR 마다 초안을 만든다. 팀의 리뷰 문체를 따른다: 한국어, 존댓말, 근거(`파일:줄`, 실패하는 입력)를 붙이고, 고칠 방향은 제안으로 쓴다. 단정하기 어려우면 질문으로 쓴다.

```markdown
### 리뷰 요약 (#N)
<무엇을 확인했는지 한두 줄, 좋았던 점 한 줄>

| # | 위치 | 심각도 | 내용 |
|---|---|---|---|
| 1 | `파일:줄` | should-fix | <한 줄> |

**제안하는 리뷰 상태**: COMMENT | REQUEST_CHANGES | APPROVE — <이유>
```

- 인라인으로 달 지적과 본문에만 둘 지적을 나눈다. 인라인은 diff 에 있는 줄에만 달 수 있다.
- 사용자에게 초안 전체를 보여 주고 **지적별로 넣을지 뺄지, 리뷰 상태를 무엇으로 할지** 고르게 한다. 리뷰 상태는 사용자가 정한다. 에이전트가 혼자 APPROVE·REQUEST_CHANGES 를 정하지 않는다.
- 고칠 것이 있으면 고쳐서 다시 보인다.

## 5. 올리기 (승인된 것만)

리뷰 하나로 묶어 올린다:
```bash
gh api repos/<owner>/<repo>/pulls/<N>/reviews -X POST --input <payload.json>
# payload: {"event": "COMMENT|REQUEST_CHANGES|APPROVE", "body": "...", "commit_id": "<head sha>",
#           "comments": [{"path": "...", "line": <n>, "side": "RIGHT", "body": "..."}]}
```
- `commit_id` 는 리뷰한 시점의 head SHA(`gh pr view <N> --json headRefOid`). 그사이 새 커밋이 올라왔으면 사용자에게 알리고 다시 볼지 묻는다.
- 인라인 줄 번호가 diff 밖이라 거절되면 그 지적을 본문으로 옮겨 다시 올린다.
- 멘션은 PR 작성자만. 목록 밖 사람을 멘션하지 않는다.

## 6. 보고

| PR | 리뷰 상태 | 올린 지적 수 (인라인/본문) | 링크 |
|---|---|---|---|

그리고 뺀 지적과 이유, 다음에 다시 볼 PR(새 커밋이 오면 다시 리뷰할 것)을 적는다. 리뷰하며 AI 쪽에 할 일이 생겼으면(계약 차이, AI 코드 수정 필요) 한 줄로 알리고 `/mm-harness:ai-feature` 나 `/mm-harness:ai-issue-reply` 로 이어 갈 수 있다고 적는다. 설명이 필요하면 `/mm-harness:teach`·`how`·`why` 를 쓸 수 있다고 한 줄 덧붙인다.
