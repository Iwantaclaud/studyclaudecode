# 머슬머슬 로직 (Muscle Muscle Logic)

네모네모로직(노노그램)을 풀면 캐릭터가 운동하고, 콤보와 점수에 따라 최종 근육량이
정해지는 캐주얼 퍼즐 게임. `../index.html`(Emerald Swarm)과 마찬가지로 의존성 없는
단일 HTML 파일이며 `muscle-muscle-logic/index.html`을 브라우저에서 열면 바로
실행된다. 빌드 도구도, 서버도 필요 없다.

## 실행 방법

`muscle-muscle-logic/index.html`을 브라우저로 열기만 하면 된다(더블클릭 또는
`file://` 경로로 직접 열어도 되고, 정적 서버로 서빙해도 된다). 인터넷 연결이 없거나
Google Fonts 요청이 막혀 있어도 시스템 기본 폰트로 정상 실행된다 — 폰트 로드
실패는 게임 기능 오류가 아니다.

## 핵심 루프

TITLE → START → 약 6~7초 컷신(매우 마른 주인공이 운동을 결심하고 덤벨을 잡는다) →
PLAYING. 로직을 푼다 → 정답 → 콤보 +1·점수 상승·캐릭터가 운동(콤보 5 이상이면
EXERCISE_HYPE로 더 열정적으로) → 연속 정답 → 오답을 내면 캐릭터가 덤벨을 발로
떨어뜨리며 시간·점수가 깎이고 콤보가 끊긴다 → 막히면 덤벨/프로틴 쉐이크/One
More! 아이템으로 정답을 미리 확인할 수 있다(제한된 개수, 점수 패널티, 콤보·시간엔
영향 없음) → 퍼즐을 완성하면 CLEAR — 캐릭터가 웃고 마지막 운동 부위가 작은 배지로
강조되며, 최종 점수·최대 콤보·시간 보너스·근육 레벨을 보여준다 → 시간이 다 되면
대신 캐릭터가 넘어지며 GAME OVER → 두 결과 화면 모두 "다시 하기"로 완전히
초기화된 새 판을 시작하는데, 이때 직전 판과 같은 퍼즐이 곧바로 반복되지 않도록
다음 퍼즐이 나머지 중에서 무작위로 뽑힌다. 정답/오답/콤보/아이템 사용/CLEAR/GAME
OVER 각각에 짧은 WebAudio 효과음이 붙어 있다.

## 개발 상태

전체 개발 계획 PHASE 1~15가 모두 완료되었다.

- [x] PHASE 1 — 기존 프로젝트(Emerald Swarm) 분석, 신규 게임으로 분리 결정
- [x] PHASE 2 — 게임 상태 구조 (TITLE → CUTSCENE → PLAYING → CLEAR / GAME_OVER)
- [x] PHASE 3 — 네모네모로직 핵심 엔진 (PuzzleManager/솔버/렌더링)
- [x] PHASE 4 — 실제 클릭/터치 입력, 정답·오답 판정, 이벤트 기반 CLEAR 연결
- [x] PHASE 5 — 실제 카운트다운 타이머, 오답 시 시간 감소, 시간 0 시 GAME OVER
- [x] PHASE 6 — 콤보 시스템(연속 정답 증가, 오답 시 초기화, 최대 콤보 기록)
- [x] PHASE 7 — 점수 시스템(콤보 보너스, 오답/아이템 패널티, CLEAR 시간 보너스)
- [x] PHASE 8 — 캐릭터 운동 시스템(정답 시 3종 운동 + 고콤보 HYPE 연출)
- [x] PHASE 9 — 오답(덤벨 낙하) 및 GAME OVER(넘어짐) 연출
- [x] PHASE 10 — 아이템 3종(덤벨/프로틴 쉐이크/One More!)
- [x] PHASE 11 — CLEAR 경험(운동별 포징, 최종 점수/최대 콤보/시간 보너스/근육 레벨 표시)
- [x] PHASE 12 — 근육량 시스템(점수 → 5단계 레벨, 결과 화면 크기/글로우 반영)
- [x] PHASE 13 — 타이틀 캐릭터 + 5컷 컷신(대략 6~7초, 자동재생+클릭+SKIP)
- [x] PHASE 14 — UI/UX 폴리시(데스크톱/640px 경계/모바일 375px 무오버플로우 확인)
- [x] PHASE 15 — 최종 통합 QA, 버그 수정(아래 참고), 코드 정리

TITLE부터 RETRY까지 전체 루프가 실제로 처음부터 끝까지 플레이 가능하다.

### PHASE 15 이후 — 완성도 폴리시 패스

PHASE 1~15 완료 후, 실제로 처음부터 플레이해보고 발견한 문제를 개선한 별도
라운드. 새 장르나 새 핵심 시스템은 추가하지 않았고 기존 컨셉 안에서만 다듬었다.

- **레이아웃/캐릭터 존재감**: 1280px 기준 캐릭터·아이템 패널이 세로로 거의
  비어 보이던 문제를 확인 — 캐릭터 스프라이트를 감싸는 원형 글로우
  "스테이지"를 추가하고 스프라이트 자체도 키웠다(`clamp(36px,9vmin,80px)`,
  스테이지는 `clamp(56px,16vmin,120px)`). 아이템 슬롯에도 은은한 카드
  배경/테두리를 줘서 아이콘이 허공에 떠 있지 않게 했고, 퍼즐 격자 최대
  크기도 420px → 480px로 키웠다.
- **CLEAR 포즈 재구성**: 기존엔 운동 부위 이모지 + 웃는 얼굴 이모지를 나란히
  이어붙여 두 개의 무관한 아이콘처럼 보였다(예: 🦵😆). 이제 얼굴(😆)이 항상
  주(主) 캐릭터로 크게 표시되고, 마지막 운동 부위는 오른쪽 아래에 작은
  원형 배지로 겹쳐 붙는다 — "웃고 있는 한 캐릭터, 강조된 신체 부위" 하나로
  읽히도록 재구성.
- **힌트 표기 수정**: 완전히 빈 줄의 힌트가 화면에 리터럴 `"0"`으로 보이던
  것을 표준 노노그램 표기대로 빈칸으로 고쳤다(내부 솔버 로직의 `[0]` 값
  자체는 그대로 유지 — 표시만 수정).
- **퍼즐 콘텐츠 확충**: 3개 → 7개. 화살표/별/스마일/트로피 4종을 추가하고
  전부 Node 스크립트로 별도 검증(구조 유효성 + 유일해)까지 마쳤다.
- **리플레이 다양성**: RESTART가 항상 같은 퍼즐로 시작하지 않도록 무작위
  선택으로 바꿨다(단, 방금 플레이한 퍼즐은 바로 다음 판에 다시 나오지
  않는다). 저장/온라인 기능은 추가하지 않았다 — 메모리 안에서만 도는 단순
  로테이션이다.
- **사운드 추가**: 파일 다운로드나 복잡한 오디오 그래프 없이, WebAudio
  오실레이터로 그때그때 생성하는 아주 짧은 효과음을 정답/오답/콤보/아이템
  사용/CLEAR/GAME OVER에 붙였다.



## 코드 구조

`index.html` 한 파일이며 역할별로 섹션 주석으로 구획되어 있다 (Emerald Swarm과 동일한 관례).

| 구획 | 역할 |
| --- | --- |
| `CONFIG` | 타이머/콤보/점수/운동/오답 연출/근육량 구간/아이템/컷신 등 모든 조정 가능한 설정값. 전부 실제 로직에서 읽힌다 |
| `EventBus` | 초소형 pub/sub. 모든 매니저가 서로·UI·GameManager를 직접 호출하지 않고 이벤트만 발행/구독한다 |
| `TimerManager` | 카운트다운(`start`/`pause`/`resume`/`stop`/`reset`, `addTime`/`subtractTime`/`setTime`). 모든 시간 변경이 `applyTime` 하나를 거치므로 0 도달 시 정지+`TIME_UP` 발행을 빠뜨릴 수 없고, `start()`가 항상 먼저 `pause()`하므로 인터벌이 중복 생성되지 않는다 |
| `ComboManager` | 연속 정답 콤보(`add`/`break`/`reset`)와 `maxCombo` 기록. 매 변경마다 `COMBO_CHANGED` 발행 |
| `ScoreManager` | 점수(`addScore`/`subtractScore`/`setScore`/`reset`/`applyTimeBonus`/`applyItemPenalty`), `CONFIG.SCORE.MIN_SCORE` 아래로 절대 내려가지 않음 |
| `MuscleManager` | 점수 → 근육 레벨(1~5) 순수 변환. 내부 상태가 없어 같은 점수는 항상 같은 레벨을 반환하고, 재시작해도 이전 레벨이 남을 수가 없다 |
| `SAMPLE_PUZZLES` | 샘플 퍼즐 7개(5×5 3종, 7×7 2종, 8×8 1종, 10×10 1종, 서로 다른 그림)의 `solution`과 담당 운동(`exercise`) 데이터. 힌트는 손으로 적지 않고 항상 solution으로부터 자동 계산된다 |
| `PuzzleRotation` | RESTART가 뽑을 다음 퍼즐을 정하는 아주 작은 헬퍼. 직전에 낸 퍼즐만 후보에서 제외하고 나머지 중 무작위로 고른다 — 저장/서버 없이 메모리 변수 하나(`lastId`)뿐 |
| `PUZZLE SOLVER` | `computeLineHints` `generateLinePatterns` `solvePuzzle` `hasUniqueSolution` `validatePuzzle` 등 순수 함수. DOM·PuzzleManager에 의존하지 않아 Node에서 단독 검증 가능 |
| `PuzzleManager` | 퍼즐 데이터, 플레이어 그리드(`UNKNOWN`/`FILLED`/`MARKED_EMPTY`), 아이템 공개 여부를 담는 별도의 `revealGrid` 보관. `isComplete`는 "플레이어가 맞혔거나 아이템으로 공개됐거나" 둘 중 하나면 그 칸을 만족으로 본다 |
| `ItemManager` | 덤벨/프로틴 쉐이크/One More! 개수와 "선택 → 대상 칸 선택 → 발동" 흐름. 공개는 `PuzzleManager.revealCell()`만 사용해 playerGrid에는 전혀 손대지 않으므로 PUZZLE_CELL_RESULT가 발생하지 않는다 — 콤보/시간에 아이템이 구조적으로 영향을 줄 수 없다 |
| `GameManager` | 최상위 상태 머신(TITLE/CUTSCENE/PLAYING/CLEAR/GAME_OVER). PLAYING 진입/이탈 시 타이머 시작/정지, CLEAR 시 시간 보너스 지급과 결과 화면 갱신, CLEAR·GAME_OVER 시 아이템 선택 취소까지 여기서 조정한다 |
| `CharacterManager` | 캐릭터 상태 머신(IDLE/EXERCISE/EXERCISE_HYPE/MISTAKE/CLEAR/GAME_OVER). 운동 중엔 현재 퍼즐의 `exercise`(이두컬/스쿼트/오버헤드 프레스)에 맞는 이모지·라벨을 보여주고, CLEAR/GAME_OVER 전환 시에는 (그 순간 화면에 실제로 보이는) 결과 화면 자체의 스프라이트(`#clearSprite`/`#gameOverSprite`)에 포즈·낙상 애니메이션을 적용한다 |
| `ExerciseManager` | 정답마다 EXERCISE(또는 콤보 5+ 시 EXERCISE_HYPE) 포즈를 잠깐 보여주고 스스로 IDLE로 되돌린다. CLEAR/GAME_OVER/MISTAKE가 먼저 끼어들면 상태 가드로 되돌리기를 건너뛴다 |
| `MistakeManager` | 오답마다 MISTAKE(덤벨 낙하) 포즈를 잠깐 보여주고 IDLE로 복귀. `GameManager.inputAllowed()`가 아니면 아예 포즈를 적용하지 않는다 — 시간을 정확히 0으로 만드는 오답처럼 같은 판정 안에서 먼저 GAME_OVER로 전환된 경우 그 낙상 포즈를 MISTAKE로 덮어쓰지 않기 위함(PHASE 15 통합 테스트로 발견·수정) |
| `UIManager` | HUD(근육 레벨/시간/콤보/점수) 및 CLEAR·GAME_OVER 화면 텍스트 동기화. 모든 값이 `TIME_UPDATED`/`COMBO_CHANGED`/`SCORE_CHANGED` 구독으로만 갱신되며 점수 변화 시 +125/-50 팝업을 띄운다 |
| `PuzzleRenderer` | PuzzleManager 상태(playerGrid + revealGrid)를 읽어 격자를 그리는 순수 렌더링 계층. `cellClass`/`cellGlyph`로 일반 칸과 아이템 공개 칸을 함께 처리 |
| `PUZZLE INPUT` | `InputMode`(채우기/X 표시 토글) + 클릭/우클릭/터치. 아이템이 선택된 상태면 같은 클릭이 일반 입력 대신 `ItemManager.useAt()`으로 라우팅된다 |
| `ITEM PANEL` | 아이템 버튼의 개수/선택/비활성 상태와 "대상 칸을 선택하세요" 안내 배너를 ItemManager 이벤트 기반으로 그린다 |
| `Cutscene` | 시작 컷신 — 5장 슬라이드 자동 진행(장당 1.3초, 총 6~7초) + 클릭으로 넘기기 + SKIP 버튼 |
| `SoundManager` | 정답/오답/콤보/아이템 사용/CLEAR/GAME OVER용 아주 짧은 WebAudio 오실레이터 비프음. 오디오 파일은 전혀 쓰지 않으며, 첫 사용자 제스처(START 클릭)에서 `AudioContext`를 미리 준비해 브라우저 자동재생 정책을 피한다 |

## 게임 상태 / 캐릭터 상태

- **게임 상태**: `TITLE → CUTSCENE → PLAYING → CLEAR | GAME_OVER`. `PLAYING`에서만
  퍼즐·아이템 입력이 허용되고(`GameManager.inputAllowed()`), 나머지 상태에서는
  전부 차단된다.
- **캐릭터 상태**: `IDLE / EXERCISE / EXERCISE_HYPE / MISTAKE / CLEAR / GAME_OVER`.
  운동/실수 포즈는 짧게 재생된 뒤 자동으로 `IDLE`로 돌아온다.

## 운동 시스템

퍼즐마다 담당 운동이 하나씩 정해져 있다(`SAMPLE_PUZZLES[*].exercise`).

| 퍼즐 | 운동 | CLEAR 배지 |
| --- | --- | --- |
| A · 다이아몬드(5×5) | SQUAT(스쿼트) | 🦵 |
| B · 덤벨(5×5) | BICEP_CURL(이두컬) | 💪 |
| C · 하트(10×10) | OVERHEAD_PRESS(오버헤드 프레스) | 🙌 |
| D · 별(7×7) | BICEP_CURL(이두컬) | 💪 |
| E · 화살표(5×5) | SQUAT(스쿼트) | 🦵 |
| F · 스마일(7×7) | OVERHEAD_PRESS(오버헤드 프레스) | 🙌 |
| G · 트로피(8×8) | SQUAT(스쿼트) | 🦵 |

정답을 맞히면 해당 운동 포즈가 `CONFIG.EXERCISE.DURATION_MS`(기본 0.6초)만큼
재생된다. 이때 콤보가 `CONFIG.COMBO.HYPE_THRESHOLD`(기본 5) 이상이면 더 빠르고
강한 EXERCISE_HYPE 연출(스케일 팝 + 글로우)로 바뀐다. CLEAR 화면에서는 웃는
얼굴(😆)이 항상 크게 표시되고, 위 배지가 오른쪽 아래에 작은 원으로 겹쳐 붙어
"마지막으로 단련한 부위"를 알려준다.

## 아이템

| 아이템 | 초기 개수 | 효과 |
| --- | --- | --- |
| 🏋️ 덤벨 | 3 | 지정한 칸을 지나는 가로+세로 전체(십자) 공개 |
| 🥤 프로틴 쉐이크 | 2 | 지정한 칸 중심 3×3(경계는 자동으로 잘림), 반경은 `CONFIG.ITEMS.PROTEIN_RADIUS` |
| 🧑‍🏫 PT쌤의 One More! | 1 | 지정한 칸 1개만 공개 |

아이템 버튼을 누르면 선택 모드로 들어가고("대상 칸을 선택하세요" 안내가 뜬다),
퍼즐 칸을 누르면 그 자리에 효과가 발동한다. 같은 버튼을 다시 누르거나 안내
배너를 탭하면 취소된다. 공개된 칸은 노란 테두리로 표시되어 플레이어가 직접
채운 칸과 구분된다. 아이템 공개는 정답 입력으로 취급되지 않으므로 콤보·시간에는
전혀 영향을 주지 않고, 점수만 사용마다 `ITEM_USAGE_PENALTY`만큼 깎인다(사용
성공 시에만). 개수는 게임 중 자동으로 채워지지 않고, 0개면 선택 자체가 막힌다.

## 점수 / 근육 레벨

- 정답 1회: `BASE_CELL_SCORE(100) + 그 시점의 콤보 × COMBO_BONUS_PER_COMBO(25)`
- 오답 1회: `-WRONG_ANSWER_PENALTY(50)`
- 아이템 사용 1회: `-ITEM_USAGE_PENALTY(25)` (성공한 경우만)
- CLEAR 순간 1회: `+남은 시간 × TIME_BONUS_PER_SECOND(10)` (GAME OVER에는 없음)
- 점수는 절대 `MIN_SCORE(0)` 아래로 내려가지 않는다

근육 레벨은 `CONFIG.MUSCLE.LEVELS`(5단계, `minScore` 기준 가장 높은 구간)로
점수에서 결정론적으로 계산된다 — 같은 점수는 언제나 같은 레벨이다.

| 레벨 | 최소 점수 | 라벨 |
| --- | --- | --- |
| 1 | 0 | 아주 마른 상태 |
| 2 | 1000 | 조금 운동한 상태 |
| 3 | 2000 | 탄탄한 상태 |
| 4 | 3000 | 근육질 |
| 5 | 4000 | 엄청난 근육질 |

CLEAR 화면의 캐릭터 스프라이트는 레벨에 따라 `muscle-lv-1`~`muscle-lv-5` 클래스로
크기와 글로우가 커진다.

## 입력 방법

- **왼쪽 클릭 / 탭**: 현재 입력 모드(기본은 "채우기") 적용, 단 아이템 선택 중에는 그 칸에 아이템 효과 발동
- **오른쪽 클릭**: 모드와 무관하게 항상 X 표시(데스크톱 전용 단축키, 아이템 선택 중엔 역시 아이템 발동). 브라우저 기본 우클릭 메뉴는 퍼즐 영역에서 막혀 있다
- **모드 토글 버튼**("채우기"/"X 표시"): 우클릭이 없는 터치 기기에서도 X 표시를 쓸 수 있게 함
- 이미 같은 상태인 칸을 다시 누르면 `UNKNOWN`으로 돌아간다. 이 "되돌리기"는 판정 대상이 아니라 시간·콤보·점수 어느 것도 변하지 않는다
- **아이템 버튼**: 누르면 선택되고(같은 버튼을 다시 누르거나 안내 배너를 탭하면 취소) 이후 퍼즐 칸을 누르면 그 자리에 효과가 발동한다

## 테스트 방법

빌드 없이 정적 파일이므로 아무 정적 서버로 서빙하거나 `file://`로 직접 열어
브라우저에서 확인한다. 개발 중에는 `window.__MML__` 디버그 훅으로 실제 UI
클릭 없이도 모든 시스템을 직접 조작·검증할 수 있다(아래 참고). 자동화
테스트(Playwright 등)를 붙일 때도 이 훅을 통해 상태를 읽고 액션을 실행하면 된다.

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
- 근육량: `getMuscleLevel`(현재 점수 기준) `getMuscleLevelForScore(score)`
- 사운드: `getSoundContextState`(AudioContext 상태) `playSound(name)`(`correct`/`wrong`/`item`/`clear`/`gameover`)

## 알려진 제한사항

- 캐릭터·아이템·연출은 여전히 이모지/CSS placeholder다. 사용자가 제공하기로 한
  캐릭터 레퍼런스 이미지가 이번 라운드에는 첨부되지 않아, 실제 스프라이트 적용은
  보류 상태다 — 대신 스테이지 글로우/크기 확대 등으로 지금 이모지 캐릭터의
  존재감만 먼저 개선했다. 이미지가 도착하면 `CHARACTER_SPRITES`/`EXERCISE_DATA`/
  아이템 아이콘 마크업만 바꿔서 교체할 수 있도록 구조는 그대로 유지했다.
- 퍼즐은 7개(5×5 3종, 7×7 2종, 8×8 1종, 10×10 1종)이고 RESTART마다 무작위
  로테이션(직전 퍼즐 제외)으로 뽑히지만, 여전히 스테이지 진행이나 난이도 곡선
  같은 개념은 없다 — 첫 판만 항상 A(다이아몬드)로 시작한다.
- 컷신은 텍스트+이모지 슬라이드 5장이며 별도 전환 애니메이션/그림은 없다.
- 사운드는 WebAudio 오실레이터로 만든 짧은 비프음뿐이며 음소거 버튼은 없다
  (요청 범위에 없었음). 브라우저가 WebAudio를 지원하지 않으면 조용히
  무시되고 게임 진행에는 영향이 없다.
