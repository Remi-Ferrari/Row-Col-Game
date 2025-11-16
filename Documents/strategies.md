# strategies.py — File Guide

## Overview and Purpose
`strategies.py` defines the **AI decision-making logic** used by computer-controlled players.  
Each strategy evaluates the current board state and selects a move based on its own heuristic.  
All strategies conform to a unified interface and are referenced dynamically via a registry dictionary.

---

## Core Data Structures and Classes

### `StrategyFunc`
- **Type:** `Callable[[BoardManager, Optional[Position]], Position]`
- **Purpose:** Defines the function signature that all strategies must implement.  
  Each strategy receives the current `BoardManager` instance and the last move (`last_pos`),  
  returning a valid `(row, col)` coordinate for the next move.

### `STRATEGY_REGISTRY`
- **Type:** `dict[str, StrategyFunc]`
- **Purpose:** Maps human-readable strategy names (e.g., `"Greedy"`) to their function objects.  
  This allows runtime selection of strategies based on configuration.

**Example:**
```python
STRATEGY_REGISTRY = {
    "Greedy": strategy_greedy_maximize,
    "MaximizeFutureMin": strategy_maximize_future_min,
    "MinimizeOpponentOptions": strategy_minimize_opponent_options,
    "PreserveHighValues": strategy_high_value_preservation,
}
```

---

## Functions

### `_best_by_key(candidates: List[Position], key_fn: Callable[[Position], Tuple], reverse: bool = True) -> Position`
Generic helper that selects the “best” candidate based on a tuple-valued metric.

- **Behavior:**  
  - Evaluates `key_fn(candidate)` for each position.  
  - Compares tuples lexicographically (primary + tie-break values).  
  - Returns the position with the highest (or lowest, if `reverse=False`) metric.

- **Purpose:**  
  Provides a consistent, reusable mechanism for ranking moves across all strategies.

- **Example:**
  ```python
  # Select maximum-value cell
  return _best_by_key(allowed, lambda p: (board.get_value(p) or 0,))
  ```

---

### `strategy_greedy_maximize(board: BoardManager, last_pos: Optional[Position]) -> Position`
Selects the allowed cell with the **highest immediate numeric value**.

- **Behavior:**  
  - Determines allowed positions (`get_all_available_positions()` if `last_pos` is `None`, else `get_allowed_positions(last_pos)`).  
  - Evaluates each by its board value.  
  - Chooses the maximum.  

- **Complexity:** `O(N)` where `N` is the number of allowed cells.  
- **Strengths:** Fast and simple baseline.  
- **Weaknesses:** Ignores future consequences.

---

### `strategy_maximize_future_min(board: BoardManager, last_pos: Optional[Position]) -> Position`
Implements a **one-step lookahead** similar to a simplified minimax approach.

- **Behavior:**  
  - For each allowed move:
    1. Temporarily removes the value at that position.
    2. Computes the opponent’s allowed positions and finds the **minimum** possible opponent gain.
    3. Restores the cell.
  - Selects the move maximizing `(my_value - opponent_min_value)`.

- **Complexity:** `O(N²)` (evaluates all opponent responses).  
- **Strengths:** Balances greed and defense.  
- **Weaknesses:** Limited to single-depth lookahead.

- **Pseudocode:**
  ```python
  best_score = -inf
  for pos in allowed:
      my_val = board.get_value(pos)
      board.board[pos[0]][pos[1]] = None
      opp = board.get_allowed_positions(pos)
      opp_min = min(board.get_value(p) or 0 for p in opp) if opp else 0
      board.board[pos[0]][pos[1]] = my_val
      score = my_val - opp_min
      if score > best_score:
          best_score, best = score, pos
  return best
  ```

---

### `strategy_minimize_opponent_options(board: BoardManager, last_pos: Optional[Position]) -> Position`
Prioritizes **board control** over raw value gain.

- **Behavior:**  
  - For each move, counts how many allowed positions the opponent would have next turn.  
  - Prefers the move that minimizes that count (fewer options = more control).  
  - Ties are resolved by higher immediate value.

- **Complexity:** `O(N²)` (requires simulating opponent visibility).  
- **Ideal Use Case:** Restricting the opponent’s freedom of movement in mid/late game.

---

### `strategy_high_value_preservation(board: BoardManager, last_pos: Optional[Position]) -> Position`
Focuses on **defense** by preserving access to high-value cells for the player while denying them to the opponent.

- **Behavior:**  
  - Determines the global maximum value still present on the board.  
  - Simulates each move and checks whether it exposes that global maximum in the opponent’s next allowed set.  
  - Prefers moves that avoid such exposure.  
  - Tie-breaks by higher immediate value.  

- **Complexity:** `O(N²)` — iterates over allowed cells and their resulting opponent options.  
- **Strengths:** Prevents catastrophic positional sacrifices.  
- **Weaknesses:** Conservative; may miss short-term gains.

---

### `get_strategy(name: str) -> StrategyFunc`
Retrieves a strategy function by its registry name.

- **Behavior:**  
  - Performs an exact dictionary lookup in `STRATEGY_REGISTRY`.  
  - Returns the function object corresponding to `name`.  
  - Raises `ValueError` if the name is unknown.

- **Purpose:**  
  Enables runtime selection of AI strategies based on user configuration or mode.

- **Example:**
  ```python
  strat = get_strategy("Greedy")
  move = strat(board, last_pos)
  ```

---

## Error Handling and Edge Cases
- **Empty allowed set:**  
  Every strategy checks for `if not allowed:` and raises  
  `ValueError("No allowed moves for strategy")` for safety.  

- **Board restoration:**  
  Strategies that simulate moves always restore the cell to its original value to maintain state consistency.  

- **Case sensitivity:**  
  Strategy lookup is exact; `"Greedy"` and `"greedy"` are treated differently.  

- **Non-destructive reads:**  
  All strategies interact with the `BoardManager` safely — they never permanently alter the board outside `remove_and_get()` calls made by the engine.

---
