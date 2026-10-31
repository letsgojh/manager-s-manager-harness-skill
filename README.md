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
| `/mm-harness:ai-feature <기능 설명>` | 설계 → 테스트 먼저 구현 → pytest → 셀프 리뷰 → develop 대상 PR 생성. PR 을 올리면 끝나고 머지는 사람이 한다 |
| `/mm-harness:ai-pr-reply [PR번호]` | PR 리뷰를 모아 지적마다 판정. 맞는 지적은 고쳐서 push 후 답글, 반박·질문 답글은 사용자 승인 후 단다 |
| `/mm-harness:grill-me` | 설계·계획만 따로 질문 라운드로 다듬는다 |

## 구성

| 구성 | 파일 | 역할 |
|---|---|---|
| 에이전트 `ai-designer` | `agents/ai-designer.md` | opus, 읽기 전용. 설계 초안·크기 판정(small/large)·사람이 정할 질문 |
| 에이전트 `ai-implementer` | `agents/ai-implementer.md` | sonnet. 태스크 하나를 TDD 로 구현·커밋. `superpowers:test-driven-development` 를 미리 로드 |
| 에이전트 `ai-reviewer` | `agents/ai-reviewer.md` | sonnet, 읽기 전용. diff 리뷰 |
| 스킬 `ai-feature` | `skills/ai-feature/` | 기능 하네스. small 은 메인 세션에서, large 는 grilling 으로 설계 확정 후 `superpowers:subagent-driven-development` |
| 스킬 `ai-pr-reply` | `skills/ai-pr-reply/` | PR 리뷰 답변. `superpowers:receiving-code-review` 원칙 |
| 스킬 `grilling`, `grill-me` | `skills/grilling/`, `skills/grill-me/` | 설계를 사용자와 질문 라운드로 정한다. 아래 출처 참고 |
| Stop 훅 | `hooks/ai-tests-gate.sh` | `ai/*.py` 가 바뀐 채 응답을 끝내면 `ai/.venv` 로 pytest. 실패하면 끝내지 못하게 막고, 같은 상태로 통과했으면 건너뛰고, 연속 3번 막으면 연다. `ai/.venv` 가 없는 저장소에서는 아무것도 하지 않는다 |

## 플러그인에 담지 못하는 설정

플러그인은 세션 기본 모델이나 권한 설정을 바꾸지 못한다. 필요하면 각자 설정에 둔다.

- 기본 모델: 에이전트 모델은 고정돼 있다. 메인 세션은 `/model sonnet` 을 권장한다 (opus 는 설계 에이전트만)
- `.env` 읽기·편집 차단은 서비스 저장소의 `.claude/settings.json` 에 있다

## 출처

- `skills/grilling`, `skills/grill-me`: [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, Copyright (c) 2026 Matt Pocock). 각 디렉터리의 `LICENSE` 참고. `grill-me` 는 플러그인 이름공간에 맞춰 호출할 스킬 이름만 바꿨다
