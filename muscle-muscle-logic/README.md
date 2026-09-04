# 머슬머슬 로직 (Muscle Muscle Logic)

네모네모로직(노노그램)을 풀면 캐릭터가 운동하고, 콤보와 점수에 따라 최종 근육량이
정해지는 캐주얼 퍼즐 게임. `../index.html`(Emerald Swarm)과 마찬가지로 의존성 없는
단일 HTML 파일이며 `muscle-muscle-logic/index.html`을 브라우저에서 열면 바로 실행된다.

## 핵심 루프

로직을 푼다 → 정답 → 콤보/점수 상승 + 캐릭터 운동(콤보 5 이상이면 열정적으로) →
퍼즐 클리어 → 남은 시간 보너스가 더해진 최종 점수로 CLEAR. 오답을 내면 캐릭터가
덤벨을 떨어뜨리는 코믹 연출과 함께 시간·점수가 깎이고 콤보가 끊긴다. 막히면 덤벨/
프로틴 쉐이크/One More! 아이템으로 정답을 미리 확인할 수 있지만(제한된 개수, 점수
패널티 있음) 콤보나 시간에는 영향을 주지 않는다. 시간이 0이 되면 GAME OVER.

## 개발 상태

전체 개발 계획은 PHASE 1(현재 프로젝트 분석) ~ PHASE 15(버그 수정)이며, 현재까지:

- [x] PHASE 1 — 기존 프로젝트(Emerald Swarm) 분석, 신규 게임으로 분리 결정
- [x] PHASE 2 — 게임 상태 구조 (TITLE → CUTSCENE → PLAYING → CLEAR / GAME_OVER)
- [x] PHASE 3 — 네모네모로직 핵심 엔진 (PuzzleManager/솔버/렌더링, 클릭 입력은 아직 없음)
- [x] PHASE 4 — 실제 클릭/터치 입력, 정답·오답 판정, 이벤트 기반 CLEAR 연결
- [x] PHASE 5 — 실제 카운트다운 타이머, 오답 시 시간 감소, 시간 0 시 GAME OVER
- [x] PHASE 6 — 콤보 시스템(연속 정답 증가, 오답 시 초기화, 최대 콤보 기록)
- [x] PHASE 7 — 점수 시스템(콤보 보너스, 오답/아이템 패널티, CLEAR 시간 보너스)
- [x] PHASE 8 — 캐릭터 운동 시스템(정답 시 3종 운동 + 고콤보 HYPE 연출)
- [x] PHASE 9 — 오답(덤벨 낙하) 및 GAME OVER(넘어짐) 연출
- [x] PHASE 10 — 아이템 3종(덤벨/프로틴 쉐이크/One More!)
- [ ] PHASE 11 — CLEAR 및 포징
- [ ] PHASE 12 — 근육량 시스템
- [ ] PHASE 13 — 타이틀 및 컷신 연출 보강 (현재는 최소 slideshow만 존재)
- [ ] PHASE 14 — UI/UX polish
- [ ] PHASE 15 — 버그 수정 및 최종 테스트

핵심 게임플레이(퍼즐 → 입력 → 판정 → 타이머/콤보/점수/캐릭터 연출/아이템 → CLEAR·
GAME OVER → 재시작)는 전부 실제로 동작한다. 남은 것은 주로 연출 폴리시(포징, 근육량
단계별 외형, 타이틀/컷신 보강)와 최종 다듬기다.

## 코드 구조

`index.html` 한 파일이며 역할별로 섹션 주석으로 구획되어 있다 (Emerald Swarm과 동일한 관례).

| 구획 | 역할 |
| --- | --- |
| `CONFIG` | 타이머/콤보/점수/운동/오답 연출/근육량 구간/아이템 관련 조정 가능한 설정값. 근육량 구간만 아직 표시 전용이고 나머지는 전부 실제 계산에 쓰인다 |
| `EventBus` | 초소형 pub/sub. 모든 매니저가 서로·UI·GameManager를 직접 호출하지 않고 이벤트만 발행/구독한다 |
| `TimerManager` | 카운트다운(`start`/`pause`/`resume`/`stop`/`reset`, `addTime`/`subtractTime`/`setTime`). 모든 시간 변경이 `applyTime` 하나를 거치므로 0 도달 시 정지+`TIME_UP` 발행을 빠뜨릴 수 없다 |
| `ComboManager` | 연속 정답 콤보(`add`/`break`/`reset`)와 `maxCombo` 기록. 매 변경마다 `COMBO_CHANGED` 발행 |
| `ScoreManager` | 점수(`addScore`/`subtractScore`/`setScore`/`reset`), `CONFIG.SCORE.MIN_SCORE` 아래로 절대 내려가지 않음. `applyTimeBonus`는 CLEAR 시 GameManager가 정확히 한 번만 호출 |
| `SAMPLE_PUZZLES` | 샘플 퍼즐 3개(5x5 초급/중급, 10x10 상급)의 `solution`과 담당 운동(`exercise`) 데이터. 힌트는 손으로 적지 않고 항상 solution으로부터 자동 계산된다 |
| `PUZZLE SOLVER` | `computeLineHints` `generateLinePatterns` `solvePuzzle` `hasUniqueSolution` `validatePuzzle` 등 순수 함수. DOM·PuzzleManager에 의존하지 않아 Node에서 단독 검증 가능 |
| `PuzzleManager` | 퍼즐 데이터, 플레이어 그리드(`UNKNOWN`/`FILLED`/`MARKED_EMPTY`), 아이템 공개 여부를 담는 별도의 `revealGrid`(§10) 보관. `setCell`은 조용한 판정, `applyPlayerInput`은 실제 플레이 경로. `isComplete`는 "플레이어가 맞혔거나 아이템으로 공개됐거나" 둘 중 하나면 그 칸을 만족으로 본다 |
| `ItemManager` | 덤벨/프로틴 쉐이크/One More! 개수와 "선택 → 대상 칸 선택 → 발동" 흐름. 공개는 `PuzzleManager.revealCell()`만 사용해 playerGrid에는 전혀 손대지 않으므로 PUZZLE_CELL_RESULT가 발생하지 않는다 — 콤보/시간에 아이템이 구조적으로 영향을 줄 수 없다 |
| `GameManager` | 최상위 상태 머신(TITLE/CUTSCENE/PLAYING/CLEAR/GAME_OVER). PLAYING 진입/이탈 시 타이머 시작/정지, CLEAR 시 시간 보너스 지급, CLEAR·GAME_OVER 시 아이템 선택 취소까지 여기서 조정한다 |
| `CharacterManager` | 캐릭터 상태 머신(IDLE/EXERCISE/EXERCISE_HYPE/MISTAKE/CLEAR/GAME_OVER). 운동 중엔 현재 퍼즐의 `exercise`(이두컬/스쿼트/오버헤드 프레스)에 맞는 이모지·라벨을 보여준다 |
| `ExerciseManager` | 정답마다 EXERCISE(또는 콤보 5+ 시 EXERCISE_HYPE) 포즈를 잠깐 보여주고 스스로 IDLE로 되돌린다. CLEAR/GAME_OVER/MISTAKE가 먼저 끼어들면 상태 가드로 되돌리기를 건너뛴다 |
| `MistakeManager` | 오답마다 MISTAKE(덤벨 낙하) 포즈를 잠깐 보여주고 IDLE로 복귀. ExerciseManager와 서로의 대기 타이머를 취소해 정답/오답이 빠르게 번갈아도 꼬이지 않는다 |
| `UIManager` | HUD(근육 레벨/시간/콤보/점수) 및 CLEAR·GAME_OVER 화면 텍스트 동기화. 모든 값이 `TIME_UPDATED`/`COMBO_CHANGED`/`SCORE_CHANGED` 구독으로만 갱신되며 점수 변화 시 +125/-50 같은 팝업을 띄운다 |
| `PuzzleRenderer` | PuzzleManager 상태(playerGrid + revealGrid)를 읽어 격자를 그리는 순수 렌더링 계층. `cellClass`/`cellGlyph`로 일반 칸과 아이템 공개 칸을 함께 처리 |
| `PUZZLE INPUT` | `InputMode`(채우기/X 표시 토글) + 클릭/우클릭/터치. 아이템이 선택된 상태면 같은 클릭이 일반 입력 대신 `ItemManager.useAt()`으로 라우팅된다 |
| `ITEM PANEL` | 아이템 버튼의 개수/선택/비활성 상태와 "대상 칸을 선택하세요" 안내 배너를 ItemManager 이벤트 기반으로 그린다 |
| `Cutscene` | 시작 컷신 — 슬라이드 자동 진행 + 클릭으로 넘기기 + SKIP 버튼. 최소 구현이며 연출은 PHASE 13에서 보강 |

## 입력 방법

- **왼쪽 클릭 / 탭**: 현재 입력 모드(기본은 "채우기") 적용, 단 아이템 선택 중에는 그 칸에 아이템 효과 발동
- **오른쪽 클릭**: 모드와 무관하게 항상 X 표시(데스크톱 전용 단축키, 아이템 선택 중엔 역시 아이템 발동). 브라우저 기본 우클릭 메뉴는 퍼즐 영역에서 막혀 있다
- **모드 토글 버튼**("채우기"/"X 표시"): 우클릭이 없는 터치 기기에서도 X 표시를 쓸 수 있게 함
- 이미 같은 상태인 칸을 다시 누르면 `UNKNOWN`으로 돌아간다. 이 "되돌리기"는 판정 대상이 아니라 시간·콤보·점수 어느 것도 변하지 않는다
- **아이템 버튼**: 누르면 선택되고(같은 버튼을 다시 누르거나 안내 배너를 탭하면 취소) 이후 퍼즐 칸을 누르면 그 자리에 효과가 발동한다

## 타이머 / 콤보 / 점수 규칙

- 남은 시간은 PLAYING 중에만 1초 단위로 감소하고 TITLE/CUTSCENE/CLEAR/GAME_OVER에서는 완전히 멈춘다. 오답 1회당 `CONFIG.TIMER.WRONG_ANSWER_PENALTY`(기본 5초)만큼 정확히 한 번 감소하며, 20초 이하 경고색·10초 이하 빨간 깜빡임(`WARNING_AT`/`CRITICAL_AT`)
- 정답은 콤보 +1, 오답은 콤보를 0으로 초기화. `maxCombo`는 오답으로도 리셋되지 않고 최고 기록을 보존한다
- 점수는 정답마다 `BASE_CELL_SCORE + 그 시점의 콤보 × COMBO_BONUS_PER_COMBO`만큼 오르고(콤보가 오른 *직후* 값을 사용), 오답마다 `WRONG_ANSWER_PENALTY`만큼, 아이템 사용마다 `ITEM_USAGE_PENALTY`만큼 내려간다 — 절대 `MIN_SCORE`(0) 아래로는 내려가지 않는다. CLEAR 순간에만 남은 시간 × `TIME_BONUS_PER_SECOND`가 정확히 한 번 더해지고, GAME OVER에는 이 보너스가 없다
- 콤보가 `CONFIG.COMBO.HYPE_THRESHOLD`(기본 5) 이상일 때 정답을 맞히면 캐릭터가 EXERCISE_HYPE로 더 열정적으로 운동한다
- 시간이 0이 되거나 퍼즐을 완성(직접 입력 또는 아이템 공개로)하면 각각 GAME_OVER/CLEAR로 자동 전환되고, 재시작 시 시간·콤보·점수·아이템 개수·퍼즐이 모두 처음 상태로 되돌아간다

## 테스트 훅

`window.__MML__`로 상태 확인과 강제 전환이 가능하다. Emerald Swarm의 `window.__ES__`와
같은 목적이며, 자동화 테스트/개발용이라 제거해도 게임 동작에는 영향이 없다.

- 게임 상태: `snap`(gameState/characterState/score/combo/maxCombo/timeLeft) `forceGameState`
  `forceCharacterState` `restart`
- 퍼즐 상태(조용한 저수준 접근): `getPuzzle` `getHints` `getPlayerGrid` `getRevealGrid`
  `isCellRevealed(row,col)` `setCell(row,col,state)` `isComplete` `resetPuzzle` `loadPuzzle(id)`
  `listPuzzles` `validatePuzzle(puzzle?)` `solvePuzzle(puzzle?)`
- 실제 입력 시뮬레이션(GameManager 게이팅 포함): `clickCell(row,col)` `rightClickCell(row,col)`
  `getCellResult(row,col)` `getInputMode` `setInputMode(mode)`
- 타이머: `getTime` `setTime(seconds)` `startTimer` `pauseTimer` `stopTimer` `forceTimeUp`
- 콤보: `getCombo` `getMaxCombo` `resetCombo`
- 점수: `getScore` `setScore(v)` `resetScore`
- 캐릭터/운동: `getCharacterState` `getCurrentExercise` `setExercise(type)` `triggerExercise`
  `triggerMistake`
- 아이템: `getItemCounts` `getSelectedItem` `selectItem(type)` `cancelItem`
  `useItem(type,row,col)`(선택+발동을 한 번에)
