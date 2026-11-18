# Current Work Log

## ASK-16 게시글 상세에 현재 사용자의 작성자 여부 제공

- 사용자 요청: 2026-11-18 `애스크티처 백엔드 ASK-16 작업하자.` Backend 개발 범위만 승인됐다.
- 공통 운영 루트 ai-dlc의 backend-issue-driven-dev 및 공통 개발/Jira/Backend 리뷰 절차를 적용한다.
- 등록 YAML 파싱·형식 검증, Backend/Specs Git 루트 및 fetch/push origin과 설정 저장소 일치를 확인했다.
- Jira https://marcel615.atlassian.net/browse/ASK-16: ASK 프로젝트의 작업 유형, area:backend, 착수 가능 상태 및 댓글 0건을 확인했다. 기존 진행/검토 중 Backend Task는 없었다. 선행 ASK-15 완료 및 공식 계약 반영을 확인했다.
- Main이 가능한 전환 21을 확인하고 진행 중으로 전환한 뒤 재조회했다. Jira 본문·완료 조건은 변경하지 않는다.
- 이전 Backend #79 PR #80은 develop에 실제 merge됐고 docs/devlog/2026-11.md에 devlog가 반영됐다. 기존 기록은 Git 이력에 보존하고 새 작업 임시 기록으로 전환한다.
- Backend origin/develop 최신 기준 457d8fecad180b8610523fb8a2e5d99baec7eed8. 작업 브랜치 feature/issue-ASK-16-post-detail-is-author.
- Specs origin/main 최신 bb06abd9d2557a37ca19ae1bde3bc1b23f878357, 계약 병합 7e0fa2e787a9ecaaa2b7d63ed60983d0d113a017 포함. 두 commit 사이 공식 4문서 diff 없음. 원본 링크는 current-task.md에 기록한다.
- Architect 읽기 전용 설계를 Main이 승인했다. 기존 선택적 principal 전달, 작성자 fetch join, JWT 오류 및 Security cache-control 설정을 유지하고 DTO boolean isAuthor와 서비스 계산만 추가한다. 계약·DB·인증정책·운영 파일 변경 불필요.
- 작성자/비작성자/익명, 사용자별 반복 조회와 no-store, 없는글/삭제글, invalid/expired/revoked token, Redis 장애 503, 수정삭제 직접 요청 인증/인가를 검증한다. 기본 명령 .\gradlew.bat test.
- 테스트 환경: Corretto 17.0.21, 기존 test 프로필의 격리 H2/임시 업로드 디렉터리. Docker Desktop을 시작하고 이번 작업 전용 askteacher-ask16-test-redis (redis:7, 127.0.0.1:16379) 생성·PONG 확인. 기존 otboo-redis 6379는 변경하지 않았다. 이번 테스트 컨테이너는 종료 시 Main이 정리한다.
- 설계 게이트 통과. 구현·전체 테스트·독립 리뷰·devlog·PR·최종 head CI는 아직 미수행.
- Builder 초기 관련 테스트는 compileTestJava에서 새 통합 테스트의 multipart/일반 요청 builder 공통 타입 불일치로 실패했다. Spring 요청 builder의 공통 상위 타입을 사용하도록 테스트를 수정했으며 제품 동작 실패는 관찰되지 않았다. 수정·재검증 1회차를 진행한다.
- 관련 테스트 재검증 1회차는 36건 중 신규 서비스 테스트 1건의 Mockito strict stubbing 불일치로 실패했고 나머지 35건은 성공했다. 좋아요 독립성 테스트에서 작성자의 false 결과도 명시적으로 stub하도록 수정했다. 수정·재검증 2회차를 진행한다.
- 관련 테스트 수정·재검증 2회차 성공: `.\gradlew.bat test --tests '*PostServiceTest' --tests '*PostControllerTest' --tests '*PostDetailApiIntegrationTest' --tests '*AuthRedisFailureIntegrationTest'` (22초). JUnit XML 36건, 실패·오류·건너뜀 0건. 제품 코드 수정은 DTO·Service 두 파일로 한정됐다. 이어 설정된 기본 전체 테스트를 실행한다.
- 설정된 기본 `.\gradlew.bat test` 전체 성공(37초). JUnit XML 총 196건, 실패·오류·건너뜀 모두 0건이며 JaCoCo XML 생성 확인. 초기 컴파일 및 신규 mock fixture 실패를 수정한 후 최종 실행은 모두 통과했다.
- API별 검증 대응: PostServiceTest는 작성자/비작성자/익명·좋아요 독립성/기존 파일 필드/없는글, PostControllerTest는 선택적 principal·정확한 isAuthor 불리언 필드와 false 포함/no-store, PostDetailApiIntegrationTest는 실제 로그인·DB 기반 사용자별 반복 GET/첨부·기존필드/삭제·없는글404/invalid·empty·wrong-signature·expired·revoked401/직접 PATCH·DELETE 익명401·타인403·작성자성공/거부후 DB·첨부 불변, AuthRedisFailureIntegrationTest는 상세 Bearer503·익명공개200을 검증한다.
- Builder 최종 자체 확인: git diff --check 통과. 제품 diff는 PostDetailResponse와 PostService에 한정되며 Controller·Security·Repository·Entity·환경·공식 Specs·AI-DLC 운영 파일을 변경하지 않았다. 계약 변경과 추가 사용자 결정은 불필요하다. Frontend 실제 연동·배포는 제외 범위이며 미검증이다. commit·push·PR·독립 리뷰·devlog·최종 head CI는 Main 후속 단계다. 테스트 Redis lifecycle은 Main이 관리한다.
- Main 구현 게이트 통과: 승인 설계와 실제 diff를 확인했고 JUnit XML 196건 실패·오류·건너뜀0, JaCoCo XML 및 diff --check를 직접 확인했다. 보호 파일 변경 없음. 열린 PR 없음, develop 활성 ruleset은 PR 경유/삭제·force push 금지, 필수 승인 수0이다. 다음 단계는 구현 commit·push·PR·독립 리뷰다.
