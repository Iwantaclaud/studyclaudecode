# 머슬머슬 로직 (Muscle Muscle Logic)

네모네모로직(노노그램)을 풀면 캐릭터가 운동하고, 콤보와 점수에 따라 최종 근육량이
정해지는 캐주얼 퍼즐 게임. `../index.html`(Emerald Swarm)과 마찬가지로 의존성 없는
단일 HTML 파일이며 `muscle-muscle-logic/index.html`을 브라우저에서 열면 바로 실행된다.

## 핵심 루프

로직을 푼다 → 정답 → 캐릭터 운동 + 콤보/점수 상승 → 퍼즐 클리어 → 포즈 + 최종 근육량 표시.
오답을 내면 덤벨을 떨어뜨리는 코믹 연출과 함께 시간이 줄고 콤보가 끊긴다.
남은 시간이 0이 되면 게임오버.

## 개발 상태

전체 개발 계획은 PHASE 1(현재 프로젝트 분석) ~ PHASE 15(버그 수정)이며, 현재까지:

- [x] PHASE 1 — 기존 프로젝트(Emerald Swarm) 분석, 신규 게임으로 분리 결정
- [x] PHASE 2 — 게임 상태 구조 (TITLE → CUTSCENE → PLAYING → CLEAR / GAME_OVER)
- [ ] PHASE 3 — 네모네모로직 핵심 기능
- [ ] PHASE 4 — 입력 및 정답 판정
- [ ] PHASE 5 — 타이머
- [ ] PHASE 6 — 콤보
- [ ] PHASE 7 — 점수
- [ ] PHASE 8 — 캐릭터 상태 및 운동
- [ ] PHASE 9 — 오답 연출 및 게임오버
- [ ] PHASE 10 — 아이템 3종
- [ ] PHASE 11 — CLEAR 및 포징
- [ ] PHASE 12 — 근육량 시스템
- [ ] PHASE 13 — 타이틀 및 컷신 연출 보강 (현재는 최소 slideshow만 존재)
- [ ] PHASE 14 — UI/UX polish
- [ ] PHASE 15 — 버그 수정 및 최종 테스트

퍼즐/타이머/콤보/점수/아이템은 아직 실제 로직이 없다. 화면 골격과 상태 전환만
동작하며, 중앙 퍼즐 패널에는 "PHASE 3에서 구현됩니다" 안내만 표시된다.

## 코드 구조

`index.html` 한 파일이며 역할별로 섹션 주석으로 구획되어 있다 (Emerald Swarm과 동일한 관례).

| 구획 | 역할 |
| --- | --- |
| `CONFIG` | 타이머/콤보/점수/근육량 구간/아이템 초기 개수 등 조정 가능한 설정값. 대부분 아직 화면 표시에만 쓰이고 실제 계산 로직은 해당 PHASE에서 채워진다 |
| `GameManager` | 최상위 게임 상태 머신(TITLE/CUTSCENE/PLAYING/CLEAR/GAME_OVER). 상태별 화면 전환과 `inputAllowed()`로 입력 허용 여부를 관리 |
| `CharacterManager` | 캐릭터 상태 머신(IDLE/EXERCISE/EXERCISE_HYPE/MISTAKE/CLEAR/GAME_OVER). 현재는 `CHARACTER_SPRITES` 이모지 테이블로 시각화 — 실제 그림으로 교체할 때 이 테이블만 바꾸면 된다 |
| `UIManager` | HUD(근육 레벨/시간/콤보/점수) 및 CLEAR·GAME_OVER 화면 텍스트 동기화 |
| `Cutscene` | 시작 컷신 — 슬라이드 자동 진행 + 클릭으로 넘기기 + SKIP 버튼. 최소 구현이며 연출은 PHASE 13에서 보강 |

## 테스트 훅

`window.__MML__`로 상태 확인과 강제 전환이 가능하다 (`snap` `forceGameState`
`forceCharacterState` `setScore` `restart`). Emerald Swarm의 `window.__ES__`와 같은
목적이며, 자동화 테스트/개발용이라 제거해도 게임 동작에는 영향이 없다.
