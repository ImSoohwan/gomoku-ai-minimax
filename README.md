# Gomoku Minimax AI (Pygame)

Pygame 기반 오목(Gomoku) 게임과 Minimax 기반 AI를 구현한 프로젝트입니다.  
AI는 제한 깊이까지 게임 트리를 탐색한 뒤, 패턴 기반 휴리스틱 평가 함수로 보드를 점수화하여 최적 수를 선택합니다.

## Demo / 실행 파일
- Windows 실행 파일(압축): (https://drive.google.com/file/d/1I_yWgIgpoGSq1WSMB61PAGRVbgc5_sHv/view?usp=sharing)
- 실행 방법: 압축 해제 후 `gomoku_game.exe` 실행  
  ※ Windows 외 OS에서는 오류가 발생할 수 있습니다.

---

## Key Features
- **Pygame GUI**: 마우스 클릭으로 게임 진행
- **Minimax AI**: 깊이 제한 탐색 + 휴리스틱 평가
- **Alpha-Beta Pruning**: 불필요한 노드 탐색 제거로 속도 개선
- **AI 성향 조절**: 공격/수비 가중치(attack/defense weight) 제공
- **설정 파일 기반 튜닝**: `settings.txt`로 주요 파라미터 저장/로드
- **패턴 점수 테이블 분리**: `pattern_scores.txt` 기반으로 유지보수 용이
- 리소스 포함: `images/`, `sounds/`, `fonts/`

---

## Tech Stack
- Python
- Pygame


---

## How It Works

### 1) Minimax (Depth-limited)
- 현재 보드에서 가능한 수들을 생성
- 각 수를 적용한 새로운 보드로 재귀 탐색
- **깊이 제한**에 도달하면 휴리스틱으로 보드를 평가해 점수 반환
- 승패가 확정되면 매우 큰/작은 값(±1,000,000 등)을 반환하여 즉시 의사결정에 반영

대표 흐름(개념):
- MAX(AI 턴): 자식 노드 중 최대 점수 선택
- MIN(상대 턴): 자식 노드 중 최소 점수 선택

### 2) Move Generation: 탐색 공간 축소
오목은 일반적으로 “기존 돌과 무관한 먼 칸”에 두는 경우가 드뭅니다.  
따라서 **이미 놓인 돌 주변(search_range 이내)**의 빈 칸만 후보로 삼아 탐색 공간을 줄였습니다.

- `get_possible_moves(board, search_range=1)`
- `any_stone_nearby(...)`로 주변 돌 존재 여부를 확인

---

## Heuristic Evaluation (Pattern-based)
휴리스틱은 “연속된 돌 + 양끝이 열려있는지”를 기준으로 점수를 부여합니다.

예시(개념):
- 4목 + 양끝 열림 > 4목 + 한쪽 열림 > 3목 + 양끝 열림 > ...

평가 함수는 다음 원칙으로 구현했습니다.
- `evaluate_board(board, player)`로 player 관점 점수 계산
- 최종 평가는 **AI 점수 - 상대 점수** 차이로 결정 (상대의 위협을 반영하기 위함)

---

## Performance Improvements & Iterations (시행착오 기록)
이 프로젝트는 “Minimax만 구현하면 끝”이 아니라, **속도와 성능을 실제로 끌어올리는 과정**이 핵심이었습니다.

### (1) 탐색 후보 축소: “주변 수만 탐색”
초기 구현에서 전체 보드를 후보로 두면 경우의 수가 폭증하여 탐색 시간이 비현실적으로 증가했습니다.  
이를 해결하기 위해 **기존 돌 주변의 빈칸만 후보로 생성**하도록 변경했습니다.
- 결과: 탐색 시간이 유의미하게 감소하여 실사용 가능한 속도에 접근

### (2) Alpha-Beta Pruning 도입
15x15 확장 및 탐색 깊이 조절 기능을 넣으면서, 다시 속도 병목이 발생했습니다.  
대표 최적화 기법인 **알파-베타 가지치기(alpha-beta pruning)**를 적용해 “어차피 선택되지 않을 노드”의 탐색을 생략했습니다.
- 결과: 특히 중반 이후 체감 속도 개선

### (3) 공격/수비 가중치(attack/defense weight)
기본 평가는 `AI_score - Opponent_score` 형태인데, 여기서
- AI 점수에 가중치를 주면 공격적
- 상대 점수에 가중치를 주면 수비적
성향 조절이 가능합니다.

`settings.txt`에서 1~9 범위 값을 받아 0.4~1.6 정도의 가중치로 변환해 적용했습니다.
- 관찰: **수비적인 세팅이 성능 면에서 더 안정적**이었고,
  특정 세팅에서 AI vs AI가 무승부에 수렴하는 결과를 확인했습니다.

### (4) possible_moves 정렬로 가지치기 효율 향상
알파-베타는 “좋은 수를 먼저 볼수록” pruning이 더 많이 발생합니다.  
이를 위해 후보 수를 **직전 수(previous_move)와의 거리 기준으로 정렬**해,
가능성이 높은 수를 우선 탐색하도록 개선했습니다.
- 관찰: 일부 상황에서 탐색 시간이 짧아지는 케이스가 발생

### (5) 필승 패턴(삼삼/위협 복합) 보너스 점수
게임을 해보면 사람이 만드는 **열린 3-3, 열린3+막힌4** 같은 “막기 어려운 복합 위협”에 AI가 취약한 경우가 있었습니다.  
단순히 패턴 점수를 더하는 방식만으로는 복합 위협을 충분히 크게 평가하지 못했기 때문입니다.

해결:
- 패턴 등장 횟수를 `pattern_counter[num][opens]`에 기록
- 특정 조합이 충족되면 bonus score 추가
  - 예: 열린 3이 2개 이상(삼삼)
  - 열린3 + 막힌4 조합
  - 막힌4 2개 등

- 결과: **복합 위협을 더 일찍 감지하고 방어하는 경향**을 유도

---

## Settings
`settings.txt`를 통해 다음 요소를 조절할 수 있습니다(예시).
- 탐색 깊이(max depth)
- search_range
- attack_weight / defense_weight
- 기타 UI/게임 옵션

설정은 게임 내 settings screen에서 +/- 버튼으로 수정하고 파일에 저장되도록 구현했습니다.

---

## Known Limitations
- 휴리스틱 기반이므로 일부 상황에서 사람이 보기엔 “비합리적”인 수를 둘 수 있습니다.
- 완전해법(완전 탐색)이 아닌 깊이 제한 탐색이므로, 깊이를 높이면 속도 비용이 급격히 증가합니다.
- Windows 외 OS에서는 실행 파일 호환 문제가 있을 수 있습니다.

---

## Credits
- 개발: ImSoohwan
- BGM 생성: MixAudio (https://mix.audio/home)
