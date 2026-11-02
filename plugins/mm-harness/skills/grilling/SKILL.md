---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
metadata:
  source: https://github.com/mattpocock/skills/tree/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/productivity/grilling
  license: MIT (Copyright (c) 2026 Matt Pocock, LICENSE 파일 참고)
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**

**왜 정해야 하나**: <이 결정이 필요한 배경. 코드·용어를 모르는 사람도 알 수 있게 쉬운 말로. 관련 사실은 `파일:줄`·이슈 번호로>

| 선택지 | 고르면 실제로 달라지는 것 | 대가 |
|---|---|---|
| A. <이름> | <구체 예시: "PM 이 X 를 하면 Y 가 된다"> | <비용·위험·되돌리기 어려움> |
| B. ... | ... | ... |

➡️ <your recommended answer> — <추천 이유 한두 문장>

---

❓ **Q2** - ...
```

Word each question so "yes" accepts your recommended answer.

Write every question so the user can decide without reading code:
- Explain jargon the first time it appears (one clause). Prefer the project's own words over library names.
- Show consequences, not mechanisms: for each option say what the user, PM, or team will see differently.
- Keep a round to what the user can hold at once. If the frontier has more than about five questions, ask the ones that unblock the most first.
- If the user says they don't follow, re-explain that question with a concrete example before moving on. Point them to `/mm-harness:teach`, `/mm-harness:how`, `/mm-harness:why` for a deeper explanation; never run those yourself.

**Check understanding after every round.** Before asking the next round, restate in plain Korean what you understood the user decided, one line per question, including any condition or exception they added ("Q3: A 로 하되, 세부 PR 은 사용자에게 보고하지 않는다"). Ask "이렇게 이해한 게 맞나요?" in the same message as the next round, so a misunderstanding is caught before it shapes later questions. If the user's answer was ambiguous, ask about it explicitly instead of picking a reading.

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent with a fresh context window to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Before closing, give a short plain-language walkthrough of what will happen once this is built: what changes for the user and team, what does not change, and what is deliberately left for later. Then ask for confirmation. Do not act on it until the user confirms you have reached a shared understanding.

<!-- 원본에서 바꾼 곳: 질문 양식에 배경·선택지별 결과 표·추천 이유를 넣고, 쉬운 말 규칙, 라운드마다 이해 확인, 끝에 쉬운 말 요약, how·why·teach 안내를 더했다 -->
