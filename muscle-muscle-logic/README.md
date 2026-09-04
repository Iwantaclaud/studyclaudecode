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
- [x] PHASE 3 — 네모네모로직 핵심 엔진 (PuzzleManager/솔버/렌더링, 클릭 입력은 아직 없음)
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

타이머/콤보/점수/아이템은 아직 실제 로직이 없다(화면에는 표시만 됨). 퍼즐은 이제 진짜로
동작한다 — 힌트 자동 생성, 정답 판정, 클리어 판정, 유일해 검증까지 모두 실제 로직이다.
다만 셀을 클릭해서 직접 플레이하는 입력 UX는 아직 없다(PHASE 4에서 연결 예정) — 지금은
`window.__MML__`을 통해 PuzzleManager를 직접 테스트하는 구조를 우선한다.

## 코드 구조

`index.html` 한 파일이며 역할별로 섹션 주석으로 구획되어 있다 (Emerald Swarm과 동일한 관례).

| 구획 | 역할 |
| --- | --- |
| `CONFIG` | 타이머/콤보/점수/근육량 구간/아이템 초기 개수 등 조정 가능한 설정값. 대부분 아직 화면 표시에만 쓰이고 실제 계산 로직은 해당 PHASE에서 채워진다 |
| `SAMPLE_PUZZLES` | 샘플 퍼즐 3개(5x5 초급/중급, 10x10 상급)의 `solution` 원본 데이터. 힌트는 여기서 손으로 적지 않고 항상 solution으로부터 자동 계산된다 |
| `PUZZLE SOLVER` | `computeLineHints` `generateLinePatterns` `solvePuzzle` `hasUniqueSolution` `validatePuzzle` 등 순수 함수. DOM·PuzzleManager에 의존하지 않아 Node에서 단독 검증 가능 |
| `PuzzleManager` | 퍼즐 데이터·플레이어 그리드(`UNKNOWN`/`FILLED`/`MARKED_EMPTY`) 보관, `setCell`(판정 결과 반환)·`isComplete`·`reset` 제공. DOM을 전혀 건드리지 않는다 |
| `GameManager` | 최상위 게임 상태 머신(TITLE/CUTSCENE/PLAYING/CLEAR/GAME_OVER). 상태별 화면 전환과 `inputAllowed()`로 입력 허용 여부를 관리 |
| `CharacterManager` | 캐릭터 상태 머신(IDLE/EXERCISE/EXERCISE_HYPE/MISTAKE/CLEAR/GAME_OVER). 현재는 `CHARACTER_SPRITES` 이모지 테이블로 시각화 — 실제 그림으로 교체할 때 이 테이블만 바꾸면 된다 |
| `UIManager` | HUD(근육 레벨/시간/콤보/점수) 및 CLEAR·GAME_OVER 화면 텍스트 동기화 |
| `PuzzleRenderer` | PuzzleManager 상태를 읽어 열 힌트/행 힌트/격자를 그리는 순수 렌더링 계층. 셀에 `data-row`/`data-col`이 이미 있어 PHASE 4가 클릭 핸들러만 추가하면 된다 |
| `Cutscene` | 시작 컷신 — 슬라이드 자동 진행 + 클릭으로 넘기기 + SKIP 버튼. 최소 구현이며 연출은 PHASE 13에서 보강 |

## 테스트 훅

`window.__MML__`로 상태 확인과 강제 전환이 가능하다. Emerald Swarm의 `window.__ES__`와
같은 목적이며, 자동화 테스트/개발용이라 제거해도 게임 동작에는 영향이 없다.

- 게임 상태: `snap` `forceGameState` `forceCharacterState` `setScore` `restart`
- 퍼즐 상태: `getPuzzle` `getHints` `getPlayerGrid` `setCell(row,col,state)` `isComplete`
  `resetPuzzle` `loadPuzzle(id)` `listPuzzles` `validatePuzzle(puzzle?)` `solvePuzzle(puzzle?)`
