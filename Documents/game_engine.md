# game_engine.py — File Guide

## Overview and Purpose
`game_engine.py` is the **control center** of RC-Game. It coordinates the turn-based loop, enforces the row/column rule through `BoardManager`, gathers moves from either humans or AI strategies, updates scores, and determines the winner. It exposes a single public entry point—`start_game`—which runs one complete match and returns a concise summary for callers (e.g., the UI in `main.py`).

---

## Core Data Structures and Classes

### `GameConfig`
A lightweight container describing how to run a match.

- `size: int` — Board size `N` for an `N×N` grid.  
- `mode: str` — One of: `H_VS_H`, `H_VS_C`, `C_VS_C`.  
- `a_strategy: Optional[str]` / `b_strategy: Optional[str]` — Strategy names if the corresponding player is computer-controlled.  
- `seed: Optional[int]` — RNG seed for reproducible random boards.  
- `preset_board: Optional[list[int]]` — Row-major list of exactly `size*size` integers for manual boards.  
- `start_player: str` — `'A'` or `'B'`; defaults to `'A'`.

### `PlayerController`
Internal representation of each player’s control method.

- Structure: `Tuple[Literal["human","computer"], Optional[Callable[[BoardManager, Optional[Position]], Position]]]`  
- Meaning: Specifies whether the player is a human or a computer; when `"computer"`, supplies the strategy function to call.

---

## Functions

### `_parse_move_input(raw: str) -> Optional[Position]`
Parses user input for a human move.

- **Expected format:** `"Row,Col"` (1-indexed), e.g., `"2,3"`.  
- **Returns:** A 0-indexed `(r, c)` tuple on success, or `None` if the format is invalid.  
- **Notes:** Bounds and availability checks are performed later, when the engine validates the parsed coordinate against the board.

---

### `_guarded_input(prompt: str) -> Optional[str]`
Reads a line of input with global command handling.

- **Recognized commands:**  
  - `Menu` → Propagates a signal (e.g., via `KeyboardInterrupt`) that the caller uses to return to the main menu.  
  - `Exit` → Terminates the program immediately and cleanly.  
- **Returns:** The raw user string (not stripped/parsed) when a regular input is entered.  
- **Purpose:** Ensures consistent behavior for global commands at any input prompt during a game.

---

### `_clear_terminal() -> None`
Clears the console in a platform-agnostic way.

- **Behavior:** Emits the appropriate escape sequence or shell command for the current OS.  
- **Use:** Called before/after rendering to keep the game view readable.

---

### `_print_scores(a_score: int, b_score: int) -> None`
Prints the current scores in a compact, single-line format, typically just under the board.

---

### `start_game(config: GameConfig) -> dict`
Runs **one complete game session**. This is the API that `main.py` calls.

- **High-level lifecycle:**
  1. **Initialize** the `BoardManager` from `config` (size, seed, or preset).  
  2. **Build controllers** based on `mode` (`H_VS_H`, `H_VS_C`, `C_VS_C`).  
  3. Set `round_num = 0`, `last_pos = None`, `current = config.start_player`.  
  4. **Loop** until no legal moves remain for the current player:  
     - Determine **allowed positions**:  
       - If `last_pos is None` (opening), all cells are legal.  
       - Otherwise, only cells in the **same row or column** as `last_pos`.  
     - **Obtain the move**:  
       - **Human** → prompt using `_guarded_input`, then parse with `_parse_move_input`, then validate.  
       - **Computer** → invoke its strategy as `strategy(board_manager, last_pos)`; strategies accept `last_pos: Optional[Position]` and handle `None` (opening) by considering the whole board.  
     - **Apply the move**: `value = board.remove_and_get(move)`; add to the current player’s score; update `last_pos = move`; increment `round_num`.  
     - **Render**: clear screen, print round, board, and scores.  
     - **Switch player** (`A ↔ B`) and continue.  
  5. **Finalize** when no legal moves remain: compute the `"winner"` (`"A"`, `"B"`, or `"Tie"`).  

- **Return value (summary dict):**
  ```python
  {
      "a_score": int,
      "b_score": int,
      "rounds": int,
      "mode": str,
      "winner": "A" | "B" | "Tie",
  }

### Complexity Notes

- **Each turn’s legality check:** `O(N)` — scans one row and one column relative to the last move.  
- **Rendering:** `O(N²)` — draws the full grid; this cost dominates for larger boards.  
- **Strategy cost:** varies by algorithm.  
  - Greedy approaches: near `O(N)` (simple maximum search).  
  - One-step lookahead (e.g., `MaximizeFutureMin`): roughly `O(N²)` per decision due to nested evaluation of opponent moves.  

---

### Error Handling and Edge Cases

Robustness is centralized in a few predictable locations to keep the flow stable:

- **Invalid `MODE`:** detected during controller setup; raises `ValueError` with a clear explanation.  
- **Missing strategies:** in `H_VS_C` or `C_VS_C`, an undefined or misspelled strategy name triggers a `ValueError` before entering the loop.  
- **Human input validation:** after `_parse_move_input`, the engine checks for valid bounds, available cell (not `None`), and adherence to the row/column rule. Invalid entries prompt re-entry with specific feedback.  
- **No legal moves:** if a player has zero allowed positions at the start of their turn, the loop terminates immediately and computes the winner.  
- **Global commands:** `_guarded_input` recognizes  
  - `Menu` → return control to main menu,  
  - `Exit` → terminate program cleanly.  
- **Opening move:** when `last_pos = None`, no positional restriction applies; both human and AI controllers can select from all cells on the board.

---
