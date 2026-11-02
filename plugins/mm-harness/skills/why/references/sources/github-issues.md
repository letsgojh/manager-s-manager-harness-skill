# GitHub 이슈

## 이 출처에 있는 것

- 기능 요청과 버그 보고, 그 토론
- 다른 파트(BE·FE)의 요청과 "선행 이슈" 관계
- ai-feature 하네스가 남긴 `## 설계 확정 (AI 하네스)` 댓글. `.superpowers/specs` 가 git 밖이라 팀원이 볼 수 있는 설계 합의 기록이다
- 라벨(예: `합의됨`)과 열림·닫힘 상태, 닫은 PR

이유가 엔지니어링 밖(제품 요구, 다른 파트의 요청)에 있을 때 가장 강하다.

## 찾는 법

```bash
# 코드 기준점의 이슈 번호를 읽는다
gh issue view <number> --json number,title,body,labels,state,author,createdAt,closedAt,comments

# 키워드로 찾는다 (닫힌 것 포함)
gh issue list --state all --search '<keyword>' --json number,title,state,labels --limit 50
gh search issues --repo {owner}/{repo} '<keyword>' --json number,title,state

# PR 이 닫은 이슈
gh pr view <pr> --json closingIssuesReferences
```

검색어는 기능 이름, 핵심 심볼, 오류 문자열, 사용자에게 보이는 용어, 결정 번호를 한국어·영어로 모두 시도한다.

## 좋은 증거의 모습

- "왜 필요한가" 를 적은 이슈 본문
- 설계 확정 댓글의 질문별 결론
- 다른 파트가 남긴 요구와 그에 대한 합의 댓글
- 선행·후속 이슈 연결

## 흔한 함정

- **이슈 본문과 실제 구현의 차이.** 이슈는 구현 전에 쓰인다. 닫은 PR 과 대조한다.
- **하네스 댓글은 에이전트 요약이다.** 사람이 확인한 뒤 남긴 것이지만 문장은 에이전트가 썼다. 범위가 중요하면 그 위아래의 사람 댓글을 함께 인용한다.
- **비공개 대화.** 디스코드나 회의에서 정하고 이슈에 남기지 않은 것은 여기에 없다. 빈틈으로 적는다.

## 돌려줄 것

관련 이슈마다: 번호, 제목, 상태, 라벨, 작성자와 날짜, 이유를 담은 원문 인용과 위치(본문 또는 댓글 링크), 연결된 PR·이슈.
