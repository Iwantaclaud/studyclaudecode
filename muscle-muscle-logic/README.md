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
- [x] PHASE 4 — 실제 클릭/터치 입력, 정답·오답 판정, 이벤트 기반 CLEAR 연결
- [x] PHASE 5 — 실제 카운트다운 타이머, 오답 시 시간 감소, 시간 0 시 GAME OVER
- [x] PHASE 6 — 콤보 시스템(연속 정답 증가, 오답 시 초기화, 최대 콤보 기록)
- [ ] PHASE 7 — 점수
- [ ] PHASE 8 — 캐릭터 상태 및 운동
- [ ] PHASE 9 — 오답 연출 및 게임오버
- [ ] PHASE 10 — 아이템 3종
- [ ] PHASE 11 — CLEAR 및 포징
- [ ] PHASE 12 — 근육량 시스템
- [ ] PHASE 13 — 타이틀 및 컷신 연출 보강 (현재는 최소 slideshow만 존재)
- [ ] PHASE 14 — UI/UX polish
- [ ] PHASE 15 — 버그 수정 및 최종 테스트

점수/아이템은 아직 실제 로직이 없다(화면에는 표시만 됨). 퍼즐은 실제로 플레이할 수
있고, 이제 타이머와 콤보도 진짜로 동작한다 — PLAYING 중에만 실제 제한 시간이 1초씩
흐르고, 오답을 내면 시간이 깎이면서 콤보가 끊기며, 정답을 연속으로 맞히면 콤보가
올라간다. 시간이 0이 되면 GAME OVER, 퍼즐을 다 풀면 CLEAR로 자동 전환된다.

## 코드 구조

`index.html` 한 파일이며 역할별로 섹션 주석으로 구획되어 있다 (Emerald Swarm과 동일한 관례).

| 구획 | 역할 |
| --- | --- |
| `CONFIG` | 타이머/콤보/점수/근육량 구간/아이템 초기 개수 등 조정 가능한 설정값. 점수/아이템 관련 수치는 아직 화면 표시에만 쓰이고 실제 계산 로직은 해당 PHASE에서 채워진다 |
| `EventBus` | 초소형 pub/sub. PuzzleManager·TimerManager·ComboManager 모두 UI·GameManager·서로를 직접 호출하지 않고 이벤트만 발행한다 — 앞으로 만들 ScoreManager 등도 같은 방식으로 구독하면 된다 |
| `TimerManager` | 카운트다운(`start`/`pause`/`resume`/`stop`/`reset`, `addTime`/`subtractTime`/`setTime`). 모든 시간 변경이 `applyTime` 하나를 거치므로 0 도달 시 정지+`TIME_UP` 발행을 빠뜨릴 수 없다. `start()`가 항상 먼저 `pause()`하므로 인터벌이 중복 생성되지 않는다 |
| `ComboManager` | 연속 정답 콤보(`add`/`break`/`reset`)와 `maxCombo` 기록. 매 변경마다 `COMBO_CHANGED` 발행 |
| `SAMPLE_PUZZLES` | 샘플 퍼즐 3개(5x5 초급/중급, 10x10 상급)의 `solution` 원본 데이터. 힌트는 여기서 손으로 적지 않고 항상 solution으로부터 자동 계산된다 |
| `PUZZLE SOLVER` | `computeLineHints` `generateLinePatterns` `solvePuzzle` `hasUniqueSolution` `validatePuzzle` 등 순수 함수. DOM·PuzzleManager에 의존하지 않아 Node에서 단독 검증 가능 |
| `PuzzleManager` | 퍼즐 데이터·플레이어 그리드(`UNKNOWN`/`FILLED`/`MARKED_EMPTY`) 보관. `setCell`은 조용한 판정(이벤트 없음), `applyPlayerInput`은 실제 플레이 경로로 `setCell` 후 EventBus에 결과/완료를 발행한다. DOM을 전혀 건드리지 않는다 |
| `GameManager` | 최상위 게임 상태 머신(TITLE/CUTSCENE/PLAYING/CLEAR/GAME_OVER). PLAYING 진입/이탈 시 `TimerManager.start()`/`stop()`을 호출하고, `PUZZLE_COMPLETE`/`TIME_UP` 이벤트를 구독해 CLEAR/GAME_OVER로 전환한다 |
| `CharacterManager` | 캐릭터 상태 머신(IDLE/EXERCISE/EXERCISE_HYPE/MISTAKE/CLEAR/GAME_OVER). 현재는 `CHARACTER_SPRITES` 이모지 테이블로 시각화 — 실제 그림으로 교체할 때 이 테이블만 바꾸면 된다 |
| `UIManager` | HUD(근육 레벨/시간/콤보/점수) 및 CLEAR·GAME_OVER 화면 텍스트 동기화. 시간·콤보는 더 이상 직접 들고 있지 않고 `TIME_UPDATED`/`COMBO_CHANGED` 구독으로만 갱신한다(경고/위험 시간 표시, 콤보 증가 팝 포함) |
| `PuzzleRenderer` | PuzzleManager 상태를 읽어 열 힌트/행 힌트/격자를 그리는 순수 렌더링 계층. `renderAll`(전체 재구성) 외에 클릭 한 번마다 쓰는 `renderCell`/`flashCell`(정답·오답 피드백)을 제공 |
| `PUZZLE INPUT` | `InputMode`(채우기/X 표시 토글) + 실제 클릭/우클릭/터치 이벤트를 `PuzzleManager.applyPlayerInput()`으로 연결. `GameManager.inputAllowed()`가 아니면 아무 것도 하지 않는다 |
| `Cutscene` | 시작 컷신 — 슬라이드 자동 진행 + 클릭으로 넘기기 + SKIP 버튼. 최소 구현이며 연출은 PHASE 13에서 보강 |

## 입력 방법

- **왼쪽 클릭 / 탭**: 현재 입력 모드(기본은 "채우기") 적용
- **오른쪽 클릭**: 모드와 무관하게 항상 X 표시(데스크톱 전용 단축키). 브라우저 기본
  우클릭 메뉴는 퍼즐 영역에서 막혀 있다
- **모드 토글 버튼**("채우기"/"X 표시"): 우클릭이 없는 터치 기기에서도 X 표시를 쓸 수 있게 함
- 이미 같은 상태인 칸을 다시 누르면 `UNKNOWN`으로 돌아간다(FILLED→클릭→UNKNOWN 등). 이
  "되돌리기"는 정답/오답 판정 대상이 아니라서 시간도 콤보도 변하지 않는다

## 타이머 / 콤보 규칙

- 남은 시간은 PLAYING 중에만 1초 단위로 감소하고, TITLE/CUTSCENE/CLEAR/GAME_OVER에서는
  완전히 멈춘다
- 오답 1회당 `CONFIG.TIMER.WRONG_ANSWER_PENALTY`(기본 5초)만큼 정확히 한 번 감소한다
- 남은 시간이 20초 이하면 HUD가 노란 경고색, 10초 이하면 빨간색으로 깜빡인다(수치는
  `CONFIG.TIMER.WARNING_AT`/`CRITICAL_AT`)
- 정답은 콤보 +1, 오답은 콤보를 0으로 초기화한다. `maxCombo`는 오답으로 리셋되지 않고
  최고 기록을 계속 보존한다
- 시간이 0이 되거나 퍼즐을 완성하면 각각 GAME_OVER/CLEAR로 자동 전환되고, 재시작 시
  시간·콤보·퍼즐이 모두 처음 상태로 되돌아간다

## 테스트 훅

`window.__MML__`로 상태 확인과 강제 전환이 가능하다. Emerald Swarm의 `window.__ES__`와
같은 목적이며, 자동화 테스트/개발용이라 제거해도 게임 동작에는 영향이 없다.

- 게임 상태: `snap`(now includes `combo`/`maxCombo`/`timeLeft`) `forceGameState`
  `forceCharacterState` `setScore` `restart`
- 퍼즐 상태(조용한 저수준 접근): `getPuzzle` `getHints` `getPlayerGrid` `setCell(row,col,state)`
  `isComplete` `resetPuzzle` `loadPuzzle(id)` `listPuzzles` `validatePuzzle(puzzle?)` `solvePuzzle(puzzle?)`
- 실제 입력 시뮬레이션(GameManager 게이팅 포함): `clickCell(row,col)` `rightClickCell(row,col)`
  `getCellResult(row,col)` `getInputMode` `setInputMode(mode)`
- 타이머: `getTime` `setTime(seconds)` `startTimer` `pauseTimer` `stopTimer` `forceTimeUp`
- 콤보: `getCombo` `getMaxCombo` `resetCombo`
