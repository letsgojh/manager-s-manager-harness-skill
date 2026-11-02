---
name: how
description: "\"X 는 어떻게 동작하나\", 고치기 전 코드 따라 읽기, 위치·소유·계층 질문(\"이건 어디에 둬야 하나\", \"어느 모듈이 이걸 맡나\", \"이 계층이 맞나\")에 답한다. 서브시스템 구조, 실행 흐름, 처음 보는 사람의 멘탈 모델을 설명한다. 사용자가 /mm-harness:how 로 부를 때만 쓴다. 동기·이유는 why 가 맡는다."
disable-model-invocation: true
metadata:
  source: https://github.com/michael-denyer/pstack-claude/tree/364c77d1a796a9483023e8bcb2027392e3ebd0d2/plugins/pstack/skills/how
  license: MIT (Copyright (c) 2026 Lauren Tan, Michael Denyer, LICENSE 파일 참고)
---

# How

질문: $ARGUMENTS

코드베이스를 탐색해 "X 는 어떻게 동작하나?" 에 답한다. 서브시스템에 처음 들어온 시니어 엔지니어가 쓸 만한 멘탈 모델을 세울 정도로 설명한다. 주석 달린 소스 코드처럼 읽힐 만큼 자세하게 쓰지는 않는다.

하네스(ai-feature 등) 진행 중이나 PR·설계를 보다가 사용자가 모르는 것을 물을 때 쓴다. 이 스킬은 아무것도 고치지 않는다. 진행 중이던 작업의 상태(브랜치, 파일, 단계)는 건드리지 않는다.

**사용자에게 하는 설명은 한국어로 쓴다.** 코드 식별자, 파일 경로, 명령은 원문 그대로 둔다.

## 서브에이전트

모든 서브에이전트는 `Agent` 도구로 `subagent_type: "general-purpose"` 를 띄운다. 브리프에 "파일을 쓰거나 고치지 않는다. 읽기만 한다" 를 넣는다.

| 역할 | 모델 |
|---|---|
| 탐색자 (explorer) | `sonnet` |
| 설명자 (explainer) | `sonnet` |

`Agent` 도구가 모델 이름을 거부하면 같은 계열에서 가장 가까운 이름을 쓰고 사용자에게 알린다.

## 0. 지도 먼저 보기

질문이 `ai/` 에 관한 것이면 `ai/CLAUDE.md` 의 "코드 지도" 와 "지식이 있는 곳" 을 먼저 읽고, 어느 모듈·파일에서 시작할지 정한다. 다른 파트면 그 파트의 `CLAUDE.md` 나 `README.md` 를 먼저 본다. 정한 시작 위치를 서브에이전트 브리프에 넣는다.

## 1. 복잡도 판단

범위가 애매하면 어떻게 해석했는지 한 줄로 말하고 탐색을 시작한다. 사용자가 방향을 바꿀 수 있다.

- **단순** (모듈 하나, 작은 유틸리티, "함수 X 는 어떻게 동작하나" 같은 좁은 질문): 탐색자를 띄우지 않는다. `references/explainer-prompt.md` 에서 "탐색자 결과" 절을 뺀 프롬프트로 설명자 하나를 띄워, 탐색과 설명을 한 번에 하게 한다. 4단계로 간다.
- **복잡** (여러 파일·서비스에 걸친 서브시스템, 여러 곳을 가로지르는 기능, 전체 구조 개요): 2단계로 간다.

애매하면 단순 경로를 고른다.

## 2. 탐색 (복잡할 때만)

질문을 2~4개의 관점으로 나눈다. 관점마다 서브시스템의 서로 다른 단면을 맡는다. `references/explorer-prompt.md` 에 관점을 채운 탐색자들을 한 메시지에서 모두 띄워 동시에 돌린다.

## 3. 종합 (복잡할 때만)

탐색자가 모두 돌아오면 `references/explainer-prompt.md` 에 모든 탐색자의 결과를 채워 설명자 하나를 띄운다.

## 4. 전달

설명자의 결과를 사용자에게 보여준다. 대화 맥락을 더하거나 읽기 쉽게 가볍게 다듬는 것은 괜찮다. 크게 다시 쓰지는 않는다.

## 출력 형식

`references/explainer-prompt.md` 에 정한 절을 쓰고, 해당 없는 절은 뺀다: 개요, 핵심 개념, 동작 방식, 어디에 있나, 주의할 점.

<!-- 원본에서 바꾼 곳: 사용자가 /mm-harness:how 로 부를 때만 동작하도록 disable-model-invocation 을 더했다. Codex·Copilot·Muse 플랫폼 안내, pstack-models.md 덮어쓰기 시트·setup-pstack·poteto-mode·Reasoning effort 절을 없애고 모델을 이 플러그인 관례(탐색자·설명자 sonnet)로 고정했다(원본 기본값은 둘 다 opus). Cursor 의 readonly 인자 대신 브리프에 읽기 전용을 적게 했다. ai/CLAUDE.md 코드 지도를 먼저 보는 0단계, 한국어 설명 규칙, 하네스 작업 상태를 건드리지 않는다는 줄을 더했다. 본문과 references 를 한국어로 옮겼다. -->
