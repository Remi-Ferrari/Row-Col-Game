# board_manager.py — File Guide

## Overview and Purpose

`board_manager.py` owns everything related to the **board state**. It creates the grid, validates and queries positions, enforces the row/column move rule, removes values, and renders the board in plain and boxed ASCII. It deliberately contains **no game-loop logic**, making it easy to test and reuse. Generally put, the file manages the entire game board system. It creates and maintains the grid, checks which positions are valid or available, applies the movement rule (same row or column), removes numbers when chosen, and prints the board for players to see.

---

## Core Data Structures and Classes

### `Position`

-   **Type:** `Tuple[int, int]`
-   **Meaning:** 0-indexed `(row, col)` coordinate used across the program.

### `BoardManager`

Encapsulates all board operations and state.

-   `size: int` — Dimension `N` for an `N×N` board.
-   `random: random.Random` — Seeded RNG used for reproducible random boards.
-   `board: List[List[Optional[int]]]` — 2D matrix; `None` marks a removed cell.
-   `last_removed_pos: Optional[Position]` — Coordinate of the **most recent** removal (for display as `X`).

---

## Functions

### `__init__(size: int, seed: Optional[int] = None, preset_board: Optional[List[int]] = None) -> None`

Constructs a new board.

-   **Behavior:**
    
    -   If `preset_board` is provided, validates it and lays values row-major into an `N×N` matrix.
    -   Otherwise, creates an `N×N` grid of random integers in `[1, 9]` using an RNG seeded by `seed` (if given).
    -   Resets `last_removed_pos = None`.
-   **Raises:**
    
    -   `ValueError` if `size < 2`.
    -   `ValueError` if `preset_board` is given but not exactly `size*size` values.
-   **Notes:**
    
    -   All internal coordinates are 0-indexed.
    -   Random boards are deterministic when `seed` is provided.

---

### `_init_board(preset_board: Optional[List[int]]) -> None`

Internal helper to populate `self.board`.

-   **Behavior:**
    
    -   When `preset_board` is not `None`, validates length and maps values row-major.
    -   Otherwise, fills with RNG values `1..9`.
-   **Raises:**
    
    -   `ValueError` for incorrect preset length.
-   **Side Effects:**
    
    -   Overwrites `self.board` in place.

---

### `in_bounds(pos: Position) -> bool`

Checks whether a position is inside the board.

-   **Returns:**
    
    -   `True` if `0 ≤ pos[0] < size` and `0 ≤ pos[1] < size`; else `False`.
-   **Complexity:**
    
    -   `O(1)`.

---

### `is_available(pos: Position) -> bool`

Verifies that a position is valid **and** not removed.

-   **Returns:**
    
    -   `True` if `in_bounds(pos)` and `self.board[r][c] is not None`.
-   **Complexity:**
    
    -   `O(1)`.

---

### `get_value(pos: Position) -> Optional[int]`

Reads the current value at a position without modifying the board.

-   **Returns:**
    
    -   The integer value at `pos`, or `None` if removed/out-of-bounds.
-   **Complexity:**
    
    -   `O(1)`.
-   **Notes:**
    
    -   This method is safe for probing during strategy simulations.

---

### `remove_and_get(pos: Position) -> int`

Consumes a cell: returns its value and marks it as removed (`None`).

-   **Behavior:**
    
    -   Validates bounds and availability.
    -   Stores the numeric value, sets that cell to `None`, and updates `last_removed_pos = pos`.
    -   Returns the stored numeric value.
-   **Raises:**
    
    -   `ValueError` if out of bounds or already removed.
-   **Complexity:**
    
    -   `O(1)`.
-   **Side Effects:**
    
    -   Mutates the board and `last_removed_pos`.

---

### `any_available() -> bool`

Checks if at least one non-removed cell exists.

-   **Returns:**
    
    -   `True` if any cell in `self.board` is not `None`; else `False`.
-   **Complexity:**
    
    -   `O(N²)` worst-case scan (fast in practice for small `N`).

---

### `get_all_available_positions() -> List[Position]`

Lists **every** currently selectable coordinate.

-   **Returns:**
    
    -   A list of `(r, c)` for all non-`None` cells.
-   **Complexity:**
    
    -   `O(N²)` to scan the grid.
-   **Use Cases:**
    
    -   First move (when there is no row/column restriction).
    -   Strategy evaluation that needs the full frontier.

---

### `get_allowed_positions(last_pos: Position) -> List[Position]`

Computes legal moves constrained to the **same row or column** as `last_pos`.

-   **Behavior:**
    
    -   Scans row `r0` and column `c0` from `last_pos`.
    -   Collects all positions that are within bounds and not removed.
    -   Typically deduplicates the `(r0, c0)` intersection (if still present).
-   **Returns:**
    
    -   A (possibly empty) list of legal `(r, c)` positions.
-   **Complexity:**
    
    -   `O(N)` for row + column scan.
-   **Notes:**
    
    -   Callers should pass a real position; round-0 scenarios should use `get_all_available_positions()`.

---

### `print_board() -> None`

Renders a simple, space-separated grid.

-   **Behavior:**
    
    -   Prints integers for available cells.
    -   Prints `X` **only** at `last_removed_pos`; older removals print as blanks for readability.
-   **Output:**
    
    -   Side-effect to stdout (no return).
-   **Use Cases:**
    
    -   Lightweight debugging or quick display without framing.

---

### `print_board_boxed(round_num: int = 0) -> None`

Renders a stylized, boxed board with headers.

-   **Behavior:**
    
    -   Displays a header with round and size.
    -   Draws column indices and row labels.
    -   Uses the same `X` convention for the **most recent** removed cell.
-   **Output:**
    
    -   Side-effect to stdout (no return).
-   **Notes:**
    
    -   Optimized for human readability during gameplay.

---

### `max_remaining_value() -> int`

Finds the greatest number still on the board.

-   **Returns:**
    
    -   The maximum remaining value, or `0` if the board is empty.
-   **Complexity:**
    
    -   `O(N²)` scan.
-   **Use Cases:**
    
    -   Heuristics (e.g., avoid exposing the global maximum).

---

## Error Handling and Edge Cases

Small, predictable failures are surfaced early with explicit exceptions.

-   **Preset validation:** `__init__` / `_init_board` raise `ValueError` if the preset length is not `size*size`.
-   **Removal safety:** `remove_and_get` raises `ValueError` for out-of-bounds or already-removed cells.
-   **Display guarantee:** rendering methods show **exactly one** `X` (the most recent removal), keeping the board readable on larger grids.
-   **Bounds discipline:** helper methods (`in_bounds`, `is_available`) centralize checks to prevent scattered off-by-one errors.

---