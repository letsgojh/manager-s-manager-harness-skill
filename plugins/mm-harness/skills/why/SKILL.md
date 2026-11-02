---
name: why
description: "\"X 는 왜 이렇게 동작하나\", \"왜 Y 를 골랐나\", 설계 근거, 회귀, 장애 후속, 근거 있는 임계값 질문에 답한다. 이 저장소의 결정 기록·계약·GitHub 이슈와 PR·git 이력을 우선 출처로, 쓸 수 있는 MCP(Notion 등)까지 출처별로 동시에 조사해 결정과 트레이드오프를 근거와 확신 수준을 붙여 돌려준다. 사용자가 /mm-harness:why 로 부를 때만 쓴다. 실행 동작은 how 가 맡는다."
disable-model-invocation: true
metadata:
  source: https://github.com/michael-denyer/pstack-claude/tree/364c77d1a796a9483023e8bcb2027392e3ebd0d2/plugins/pstack/skills/why
  license: MIT (Copyright (c) 2026 Lauren Tan, Michael Denyer, LICENSE 파일 참고)
---

# Why

질문: $ARGUMENTS

코드 뒤에 있는 동기와 의도를 조사한다. 짝인 `how` 스킬은 코드가 무엇을 하고 어떻게 동작하는지에 답한다. `why` 는 어떤 힘이 코드를 지금 모양으로 만들었는지에 답한다.

하네스(ai-feature 등) 진행 중이나 PR·설계를 보다가 사용자가 모르는 것을 물을 때 쓴다. 이 스킬은 아무것도 고치지 않는다. 진행 중이던 작업의 상태(브랜치, 파일, 단계)는 건드리지 않는다.

**사용자에게 하는 설명은 한국어로 쓴다.** 인용은 원문 그대로 두고, 코드 식별자, 파일 경로, 명령도 원문 그대로 둔다.

## 서브에이전트

모든 서브에이전트는 `Agent` 도구로 `subagent_type: "general-purpose"` 를 띄운다. MCP 도구를 써야 하므로 읽기 전용 에이전트 타입(`Explore` 등)을 쓰지 않는다. 대신 브리프에 "파일을 쓰거나, 커밋하거나, 외부 상태(이슈·PR·Notion 등)를 바꾸지 않는다" 를 넣는다.

| 역할 | 모델 |
|---|---|
| 조사자 (investigator) | `sonnet` |
| 합성자 (synthesizer) | `opus` |

합성자는 증거마다 확신 등급을 매기고 모순을 가려내는, 판단이 큰 역할이라 `opus` 를 쓴다. `Agent` 도구가 모델 이름을 거부하면 같은 계열에서 가장 가까운 이름을 쓰고 사용자에게 알린다.

## 자세

**신중하고 조심스럽고 정확한 조사자**로 일한다. 아는 것과 추론한 것을 정직하게 나눈다. 확신 체계와 표현 지침은 `references/epistemics.md` 에 있다. 합성자는 반드시 이를 따른다.

## 1단계. 대상과 질문 파악

**대상**은 대개 코드 조각, 패턴, 기능, 이름 붙은 설계 결정이다. **질문**은 대개 설계 근거, 트레이드오프, 동기가 된 엣지 케이스, 외부 제약, 죽은 코드, 넓은 이력 훑기다.

대상이 모호하면("이거 왜 이렇게 해?" 인데 가리키는 것이 분명하지 않으면) 대화 맥락(열린 파일, 최근 수정, 방금 이야기한 것, 진행 중인 하네스 단계)으로 가장 그럴듯하게 짐작한다. 해석을 짧게 말해 사용자가 바로잡을 수 있게 하고 진행한다.

## 2단계. 코드 기준점 세우기

조사자를 띄우기 전에 메인 세션에서 기준점을 만든다: 파일 경로와 줄 범위, 핵심 심볼(함수, 클래스, 상수), 대상을 건드린 최근 커밋 몇 개, 머지 커밋에서 뽑은 PR 번호(제목의 `(#1234)` 패턴), 연결된 이슈 번호, 대상 근처 코드 주석이 가리키는 결정 기록 번호(예: "결정 기록 0013", "0026").

- `git blame -L <start>,<end> <file>` 로 마지막으로 건드린 커밋
- `git log --follow -p -- <file>` 로 이름 바뀜을 따라간 전체 이력
- `git log --oneline -20 -- <file>` 로 PR 번호가 보이는 최근 커밋
- `git log -1 --format=%B <commit>` 으로 PR 번호 추출
- 의미 있는 커밋마다 PR 본문과 토론: `gh pr view <number> --json title,body,author,createdAt,mergedAt,labels,closingIssuesReferences,comments,reviews`

이 기준점은 모든 조사자와 합성자에게 간다.

## 3단계. 조사자를 동시에 띄우기

**기본은 전체 병렬 조사다.** 목표는 최소한의 조사가 아니라 빠짐없는 출처 지도다. 검색을 건너뛰지 말고 빈 결과를 기록한다.

### 출처 범주

이 하네스는 Manager's Manager 저장소의 `ai/` 파트용이다. 이 저장소에서는 1~3번이 늘 쓸 수 있는 우선 출처다.

1. **저장소 결정 기록·계약** (우선 출처). 플레이북 `references/sources/project-docs.md`. `ai/docs/basis_for_decision/`(결정 기록, AI 가 쓴 것은 `ai/`, 사람이 쓴 것은 `member/`), `backend/docs/contracts/`·`frontend/docs/contracts/`(파트 간 계약), `ai/docs/request/`, `ai/docs/plan/`, 트러블슈팅 기록, git 밖 작업 파일 `.superpowers/specs/`·`.superpowers/plans/`, 코드 주석의 결정 번호. 가장 잘 찾는 것: *팀이 글로 남긴 결정과 그 이유*.
2. **소스 관리 이력** (git, `gh` 로 PR, 코드 주석, 테스트). 플레이북 `references/sources/code-archaeology.md`. 가장 잘 찾는 것: *구현과 리뷰 중에 남은 근거*.
3. **GitHub 이슈** (`gh issue`). 플레이북 `references/sources/github-issues.md`. 가장 잘 찾는 것: *제품·요청 쪽의 계기, 다른 파트의 요구, 설계 확정 댓글*.
4. **긴 문서: Notion** (Notion MCP 가 있고 인증됐을 때만). 플레이북 `references/sources/notion.md`. 팀이 Notion 을 쓴다. 가장 잘 찾는 것: *코드가 되기 전에 길게 적힌 설계 근거, 회의록*.
5. **그 밖의 MCP** (있을 때만): 실시간 채팅(Slack, Discord 등), 인프라 관측, 오류 추적, 제품 분석 저장소. 플레이북 `references/sources/other-mcp.md`. 이 프로젝트에는 보통 없다.

### 탐지

조사자를 띄우기 전에 이 환경에서 쓸 수 있는 MCP 를 확인한다. 시스템 프롬프트의 도구 목록(`mcp__<server>__<name>` 접두사)을 보고, 지연 로딩 도구 이름 목록도 본다. 이름만 있고 `authenticate` 도구만 보이는 서버는 인증되지 않은 것이므로 "쓸 수 없음" 으로 기록한다. 여러 범주에 걸치는 MCP 는 주된 증거에 맞는 범주 하나로 고르고, 애매한 경우는 출처 지도에 적는다.

범주마다 조사자 하나를 띄우고, 한 메시지에서 모두 띄워 동시에 돌린다. 조사자 하나는 출처(도구 또는 MCP) 하나만 맡는다. 한 에이전트에게 여러 MCP 를 맡기지 않는다.

조사자의 프롬프트는 `references/investigator-prompt.md` 를 채운 것에 다음을 더한다.

- 그 범주의 플레이북 `references/sources/<source>.md` (목록: `references/source-playbook.md`)
- **대상 코드가 방어 코드처럼 보이면** `references/sources/incident-postmortem.md` (null 검사, 재시도, 타임아웃, 속도 제한, 기능 플래그, 잠금, 복구 경로, 폴백)
- 코드 기준점과 사용자의 원래 질문

### 조사자를 건너뛸 때

**글로 쓴 명시적 이유**가 있을 때만 건너뛴다. 그 이유는 최종 "참고한 출처" 에 들어간다. 타당한 이유는 둘이다.

- **그 범주에 맞는 MCP 가 이 환경에 없다.** 선택이 아니라 빈틈으로 기록한다: "Notion 은 건너뜀. Notion MCP 가 인증되지 않아 긴 문서 기록을 찾을 수 없었다."
- **그 출처가 관련 없음이 증명된다.** "아마 관련 없음" 이 아니라 증명이어야 한다. 기준이 높다: "오류 추적은 건너뜀. 대상은 실행 경로가 없는 빌드 스크립트다."

1~3번은 늘 쓸 수 있으므로 건너뛰지 않는다. 커밋 하나짜리 사소한 대상이고 PR 설명이나 결정 기록이 이미 완전한 답을 담고 있으면, 나머지 범주 검색이 모두 중복임을 확인한 **뒤에만** 바로 답할 수 있다. 그렇게 했다고 분명히 말한다. 드물어야 한다.

## 4단계. 합성

합성자 하나를 띄운다. 합성자가 받는 것:

1. 조사자 결과(빈 결과, 이유와 함께 건너뛴 범주 포함)
2. 코드 기준점
3. 사용자의 원래 질문
4. `references/epistemics.md` 의 확신 체계
5. `references/synthesizer-prompt.md` 의 합성자 프롬프트 템플릿

파일은 내용을 붙이지 말고 절대 경로로 넘겨 합성자가 직접 읽게 한다.

## 5단계. 전달

합성자의 결과를 사용자에게 보여준다. 읽기 쉽게 가볍게 다듬거나 대화 맥락을 더할 수 있지만 **확신 표현은 고쳐 쓰지 않는다**.

## 출력 형식

구조는 `references/synthesizer-prompt.md` 에 있다: 질문, 대상 코드, 찾은 것, 합리적으로 추론할 수 있는 것, 경쟁하는 가설, 모르는 것, 참고한 출처, 확신 요약. 필요하면 고쳐 쓰되 확신 구분은 지키고, "참고한 출처" 는 조사자마다 한 줄씩, 아무것도 못 찾았거나 건너뛴 것까지 이유와 함께 적는다.

사용자의 `why` 질문이 이 코드를 바꾸기 위한 사전 질문이면(예: 하네스 설계 단계에서 물었으면), "참고한 출처" 뒤에 찾은 내력을 바꿀 때의 제약으로 바꿔 적는다: 지킬 것 / 바꿀 것 / 피할 것 / 위험.

## 피해야 할 실패

- **최신 편향.** 가장 최근 커밋이 권위 있다고 가정하는 것. 지금 모양은 여러 이전 결정이 쌓인 결과인 경우가 많다. 거슬러 올라간다. 결정 기록도 마찬가지다. 뒤의 기록이 앞의 기록을 바꿨는지 확인한다.

<!-- 원본에서 바꾼 곳: 사용자가 /mm-harness:why 로 부를 때만 동작하도록 disable-model-invocation 을 더했다. Codex·Copilot·Muse 플랫폼 안내, pstack-models.md 덮어쓰기 시트·setup-pstack·poteto-mode·Reasoning effort 절, Cursor 의 readonly 인자 설명을 없앴다. 모델은 조사자 sonnet, 합성자 opus 로 고정했다(원본 기본값은 둘 다 opus). 원본의 일곱 범주(소스 관리·이슈 트래커·긴 문서·채팅·관측·오류·분석)를 이 저장소에 맞게 바꿨다: 저장소 결정 기록·계약(basis_for_decision ai/member, contracts, request, plan, 트러블슈팅, .superpowers specs·plans, 코드 주석 결정 번호)을 우선 출처로 더하고, 이슈 트래커를 GitHub 이슈(gh)로, 긴 문서를 Notion(MCP 있을 때)으로 두고, 채팅·관측·오류·분석은 "그 밖의 MCP(있을 때만)" 하나로 묶었다. 그에 맞춰 Datadog·Sentry·Databricks·Linear·Slack 플레이북을 빼고 project-docs.md·github-issues.md·other-mcp.md 를 새로 썼다. 한국어 설명 규칙, 하네스 작업 상태를 건드리지 않는다는 줄, 바꾸기 전 질문일 때의 제약 목록 예시, 결정 기록 간 최신 편향 확인을 더했다. 본문과 references 를 한국어로 옮겼다. -->
