# 코드 고고학 (git + 저장소 안)

## 이 출처에 있는 것

- 커밋 이력 (메시지, 날짜, 작성자, diff)
- PR 설명, 리뷰 댓글, 토론 스레드 (`gh` 로)
- 코드 주석, TODO, FIXME, 폐기 안내
- 테스트. 테스트 이름과 단언은 변경의 동기가 된 엣지 케이스를 담는 일이 많다
- 같은 커밋에서 함께 바뀐 파일 (함께 바뀜 신호)
- 커밋 메시지와 PR 본문의 이슈 번호, 결정 기록 번호

코드에 바로 붙어 있어 가장 믿을 만하고 가장 완전한 출처다. 저장소를 거친 것은 모두 여기 있다. 결정 기록 문서 자체의 내용은 "저장소 결정 기록·계약" 조사자가 맡는다. 여기서는 그 문서를 가리키는 커밋·PR·주석까지만 보고 "추가 단서" 에 적는다.

## 찾는 법

시작 커밋 목록을 넓힌다.

```bash
# 이름 바뀜을 따라간 이력
git log --follow --oneline -- <file>

# 픽액스: 이 문자열을 더하거나 뺀 커밋
git log -S '<exact_string_from_code>' -- <file>

# 패턴으로
git log -G '<regex>' -- <file>

# 줄마다 누가 언제
git blame -L <start>,<end> <file>

# 커밋 하나의 전체 diff
git show <hash>

# 두 지점 사이에 이 파일을 건드린 커밋
git log <old>..<new> -p -- <file>
```

PR 을 읽는다.

```bash
gh pr view <number> --json title,body,author,createdAt,mergedAt,labels,closingIssuesReferences,comments,reviews
# 줄 단위 리뷰 댓글
gh api repos/{owner}/{repo}/pulls/<number>/comments
# 키워드로 PR 찾기
gh pr list --state all --search '<keyword>' --json number,title,mergedAt
```

주석과 테스트를 본다.

```bash
# 대상 근처의 TODO·FIXME
rg -n -C2 '(TODO|FIXME|HACK|XXX|NOTE)' <target_file>

# 관련 테스트 (ai/ 는 ai/tests/)
rg -l '<symbol>' ai/tests
```

## 흔한 함정

- **스쿼시 머지.** 커밋 메시지가 PR 제목뿐일 수 있다. PR 본문과 리뷰를 꼭 연다.
- **하네스가 만든 PR.** ai-feature 하네스 PR 본문에는 설계 요약과 고치지 않은 리뷰 지적이 있다. 이것도 에이전트가 쓴 글이므로 사람의 리뷰 댓글과 구분한다.
