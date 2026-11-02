# 저장소 결정 기록·계약 (우선 출처)

## 이 출처에 있는 것

Manager's Manager 저장소는 "저장소에 없는 지식은 에이전트에게 없는 것과 같다" 는 원칙으로 결정을 글로 남긴다(`ai/CLAUDE.md` 의 "지식이 있는 곳"). 이유를 찾을 때 가장 먼저 볼 곳이다.

| 무엇 | 어디 |
|---|---|
| 결정 기록과 진행 로그 | `ai/docs/basis_for_decision/`. 결정 기록은 `YYYY-MM-DD-NNNN-*.md`(번호 있음), 진행 로그는 `YYYY-MM-DD-*.md`. **AI 가 쓴 것은 `ai/`, 사람이 쓴 것은 `member/`**. 다른 파트도 `<part>/docs/basis_for_decision/` 이 있을 수 있다 |
| 파트 간 계약 | `backend/docs/contracts/`(예: `backend-ai-contract.md`), `frontend/docs/contracts/`(예: `frontend-api-contract.md`) |
| 다른 파트의 요청 | `ai/docs/request/` |
| 목표·마일스톤 | `ai/docs/plan/` (보호 문서) |
| 트러블슈팅 기록 | `*/docs/plan/*/trouble-shooting/` (사람이 씀) |
| 문서 양식·규칙 | `docs/README.md`, `docs/agent-rules.md`, `ai/docs/README.md` |
| 전체 흐름 | `docs/파트별 전체 흐름.md`, `docs/*.excalidraw*` |
| git 밖 작업 파일 | `.superpowers/specs/`(합의된 설계), `.superpowers/plans/`(구현 계획). 저장소 루트에 있지만 git 에 올라가지 않는다. 그 체크아웃에서만 보이고 다른 사람은 못 본다 |
| 코드 주석의 결정 번호 | 주석이 "결정 기록 0013", "(0026)" 처럼 번호로 가리킨다. 번호로 `*/docs/basis_for_decision/` 에서 파일을 찾는다 |

## 찾는 법

```bash
# 코드 주석이 가리키는 결정 번호를 모은다
rg -n '결정 기록 ?[0-9]{4}|\b0[0-9]{3}\b' <target_file>

# 번호로 결정 기록 찾기 (ai/ 와 member/ 모두)
ls */docs/basis_for_decision/*/ | rg -- '-<NNNN>-'

# 키워드로 결정 기록·계약·요청·트러블슈팅 훑기
rg -n -i '<symbol|기능 이름|임계값>' ai/docs backend/docs/contracts frontend/docs/contracts
rg -l -i '<keyword>' -g '**/trouble-shooting/**'

# git 밖 작업 파일 (있을 때만)
ls .superpowers/specs .superpowers/plans 2>/dev/null
rg -n -i '<keyword>' .superpowers 2>/dev/null

# 결정 기록이 언제, 어떤 커밋으로 들어오고 바뀌었는지
git log --follow --oneline -- <decision_record_path>
```

결정 기록은 끝까지 읽는다. "고려한 대안", "결과", "후속" 같은 절에 핵심이 있는 일이 많다. 한 결정이 다른 결정을 대체하거나 고쳤는지 본문의 참조와 날짜로 확인한다.

## 좋은 증거의 모습

- 대상 코드의 목적과 맞는 "맥락", "결정", "이유" 절이 있는 결정 기록
- "고려한 대안" 이나 "택하지 않은 방식" 절
- 그 필드·동작을 정하는 계약 조항과, 그 조항의 합의 상태
- 다른 파트 요청 문서에 적힌 요구
- 대상 코드를 고친 원인과 해결을 적은 트러블슈팅 기록

## 흔한 함정

- **AI 가 쓴 기록을 사람의 결정으로 읽기.** `ai/` 기록의 "사용자가 정했다" 는 에이전트 요약이다. 범위가 중요하면 `member/` 기록, 이슈 댓글, PR 리뷰에서 원래 말을 찾는다. 찾으면 둘을 함께 보고한다.
- **계획과 실제의 차이.** `.superpowers/specs`·`plans` 와 진행 로그는 구현 전에 쓰여 나중에 바뀌었을 수 있다. 실제 PR·코드와 대조하고 차이를 적는다.
- **계약의 합의 상태.** 계약 문서는 구현 사실과 합의 상태를 나눠 적는다. 합의되지 않은 조항을 결정으로 보고하지 않는다.
- **오래된 결정.** 뒤의 결정 기록이 앞의 것을 바꿨을 수 있다. 같은 주제의 기록을 날짜순으로 모두 본다.
- **git 밖 파일이 없음.** `.superpowers/` 가 없거나 비어 있으면 빈틈으로 적는다(다른 체크아웃에만 있을 수 있다).

## 돌려줄 것

관련 문서마다: 경로, 작성 주체(`ai/` 또는 `member/`, 트러블슈팅 작성자), 날짜, 번호, 이유를 담은 원문 인용과 절 위치, 그 문서가 가리키는 다른 결정·이슈·PR, 대체되었거나 계획 단계인지 여부.
