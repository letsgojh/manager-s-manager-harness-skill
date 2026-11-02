# 출처 플레이북

why 스킬은 쓸 수 있는 출처 범주마다 조사자 하나를 띄우고, 조사자마다 아래 플레이북 하나를 읽힌다.

| 범주 | 플레이북 | 언제 |
|---|---|---|
| 저장소 결정 기록·계약 (우선 출처) | [`project-docs.md`](./sources/project-docs.md) | 늘 |
| 소스 관리 이력 | [`code-archaeology.md`](./sources/code-archaeology.md) | 늘 (git, `gh`) |
| GitHub 이슈 | [`github-issues.md`](./sources/github-issues.md) | 늘 (`gh`) |
| 긴 문서: Notion | [`notion.md`](./sources/notion.md) | Notion MCP 가 있고 인증됐을 때만 |
| 그 밖의 MCP (채팅, 인프라 관측, 오류 추적, 제품 분석) | [`other-mcp.md`](./sources/other-mcp.md) | 맞는 MCP 가 있을 때만, MCP 마다 조사자 하나 |

범주를 가로지르는 것:

- [`incident-postmortem.md`](./sources/incident-postmortem.md). 대상 코드가 방어 코드처럼 보이면(null 검사, 재시도, 타임아웃, 속도 제한, 기능 플래그, 잠금, 복구 경로, 폴백) 더한다.
