## docker Jenkins 에 PollSCM 활용

### Project > Configure > Build Triggers
- Build periodically -> cron job
  - 수정사항이 없어도 빌드 수행 
- Poll SCM -> cron job
  - 커밋된 내용이 있어야만 빌드 수행

> 체크박스 선택 : Build whenever a SNAPSHOT dependency is built
> 체크박스 선택 : Poll SCM
> Schedule (*(분) *(시) *(일) *(월) *(요일)) : * * * * * (매 초마다 수행)