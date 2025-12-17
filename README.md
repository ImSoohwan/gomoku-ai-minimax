📄 한국어 문서는 README_KR.md를 참고하세요.

📝 Detailed development notes are available on my blog: 
* Part1: https://blog.naver.com/d_soohwan/223877512044
* Part2: https://blog.naver.com/d_soohwan/223887231185

---
# Gomoku Minimax AI (Pygame)

This project implements a Pygame-based Gomoku game with a Minimax-based AI.  
The AI explores the game tree up to a limited depth, evaluates the board using a pattern-based heuristic evaluation function, and selects the optimal move.

## Demo / Executable
- Windows executable (ZIP): (https://drive.google.com/file/d/1I_yWgIgpoGSq1WSMB61PAGRVbgc5_sHv/view?usp=sharing)
- How to run: Extract the archive and run `gomoku_game.exe`  
  ※ Errors may occur on operating systems other than Windows.

---

## Key Features
- **Pygame GUI**: Play the game via mouse clicks
- **Minimax AI**: Depth-limited search with heuristic evaluation
- **Alpha-Beta Pruning**: Improves performance by eliminating unnecessary node exploration
- **AI Behavior Tuning**: Adjustable attack/defense weights
- **Config-based Tuning**: Save/load key parameters via `settings.txt`
- **Separated Pattern Score Table**: Easy maintenance using `pattern_scores.txt`
- Included resources: `images/`, `sounds/`, `fonts/`

---

## Tech Stack
- Python
- Pygame

---

## How It Works

### 1) Minimax (Depth-limited)
- Generate all possible moves from the current board state
- Recursively explore new boards created by applying each move
- When the **depth limit** is reached, evaluate the board using a heuristic function and return a score
- If a win or loss is determined, return a very large/small value (e.g., ±1,000,000) to immediately influence decision-making

Conceptual flow:
- MAX (AI turn): Choose the child node with the maximum score
- MIN (Opponent turn): Choose the child node with the minimum score

### 2) Move Generation: Reducing the Search Space
In Gomoku, placing a stone in a position far away from existing stones is rarely meaningful.  
Therefore, the search space is reduced by considering only **empty cells near already placed stones (within `search_range`)**.

- `get_possible_moves(board, search_range=1)`
- Check for nearby stones using `any_stone_nearby(...)`

---

## Heuristic Evaluation (Pattern-based)
The heuristic assigns scores based on **consecutive stones and whether the ends are open**.

Example (conceptual):
- Open four > four with one blocked end > open three > ...

The evaluation function follows these principles:
- `evaluate_board(board, player)` calculates the score from the given player’s perspective
- The final evaluation is determined by **AI score - opponent score**, so that opponent threats are taken into account

---

## Performance Improvements & Iterations (Trial-and-Error Log)
This project was not just about implementing Minimax, but about **actually improving speed and performance through iteration**.

### (1) Candidate Reduction: “Search Only Nearby Moves”
In the initial implementation, considering the entire board as candidates caused a combinatorial explosion and unrealistic search times.  
This was resolved by generating candidates **only from empty cells near existing stones**.
- Result: Search time was significantly reduced, reaching a practically usable speed

### (2) Introducing Alpha-Beta Pruning
After expanding to a 15×15 board and adding adjustable search depth, performance bottlenecks reappeared.  
By applying **alpha-beta pruning**, exploration of nodes that would never be chosen was skipped.
- Result: Noticeable speed improvements, especially in the mid-to-late game

### (3) Attack/Defense Weights
The base evaluation is `AI_score - Opponent_score`, where:
- Increasing the weight on the AI score makes the AI more aggressive
- Increasing the weight on the opponent score makes the AI more defensive

Values from 1 to 9 are read from `settings.txt` and mapped to weights roughly in the range of 0.4 to 1.6.
- Observation: **Defensive settings were more stable in terms of performance**, and in some configurations, AI vs AI games converged toward draws

### (4) Sorting possible_moves to Improve Pruning Efficiency
Alpha-beta pruning becomes more effective when “good moves are explored first.”  
To achieve this, candidate moves are **sorted by their distance from the previous move**, prioritizing more promising positions.
- Observation: In some situations, this noticeably reduced search time

### (5) Bonus Scores for Forced-Win Patterns (Double-Three / Combined Threats)
During gameplay, the AI was sometimes vulnerable to human-created **complex threats** such as open double-threes or open-three + blocked-four combinations.  
Simple additive pattern scoring was not sufficient to properly value these compound threats.

Solution:
- Track pattern occurrences in `pattern_counter[num][opens]`
- Add bonus scores when specific combinations are satisfied
  - Examples: two or more open threes (double-three)
  - open-three + blocked-four combination
  - two blocked-fours

- Result: The AI tends to **detect and defend against compound threats earlier**

---

## Settings
The following elements can be adjusted via `settings.txt` (examples):
- Search depth (max depth)
- search_range
- attack_weight / defense_weight
- Other UI/game options

Settings can be modified using +/- buttons in the in-game settings screen and saved to the file.

---

## Known Limitations
- Since the AI is heuristic-based, it may sometimes make moves that appear “irrational” to human players
- Because the search is depth-limited rather than exhaustive, increasing depth causes a rapid increase in computational cost
- Executable compatibility issues may occur on operating systems other than Windows

---

## Credits
- Development: ImSoohwan
- BGM generation: MixAudio (https://mix.audio/home)
