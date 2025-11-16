# config.txt — File Guide

## Overview and Purpose
`config.txt` is the **configuration backbone** of the Row-Column Game.  
It stores all persistent settings that define how the game behaves between runs — including board size, game mode, player strategies, and random seed information.  
When the game starts, `main.py` reads this file to automatically set up the environment, allowing players to skip repetitive menu inputs.  
It can be edited manually with any text editor or updated safely from within the in-game configuration interface.  

This file is simple and human-readable, designed so non-programmers can easily tweak gameplay parameters without touching the source code.

---

## File Structure and Keys

Each line follows a `KEY=VALUE` format.  
Keys map directly to attributes in the program’s internal `GameConfig` class and some temporary UI state used by the configuration menu.

| Key | Type / Values | Required | Description |
|-----|----------------|-----------|--------------|
| **SIZE** | Integer | Yes | Defines board dimensions `N×N` (must be ≥ 2). |
| **MODE** | String | Yes | Game mode: `H_VS_H` (Human vs Human), `H_VS_C` (Human vs Computer), or `C_VS_C` (Computer vs Computer). |
| **A_STRATEGY** | String | Conditional | Strategy name for Player A if A is a computer. |
| **B_STRATEGY** | String | Conditional | Strategy name for Player B if B is a computer. |
| **START_PLAYER** | `A` or `B` | No | Determines who begins the game; defaults to `A`. |
| **BOARD_SOURCE** | `RANDOM` or `MANUAL` | No | Determines whether the board is generated randomly or manually defined. |
| **SEED** | Integer | Optional | Random number generator seed (used only when `BOARD_SOURCE=RANDOM`). |
| **BOARD** | Comma-separated integers | Optional | Exactly `SIZE×SIZE` values (row-major order). Used when `BOARD_SOURCE=MANUAL`. |

---

## How It’s Used by the Program

### Reading (`load_config`)
When the game launches, `main.py` loads `config.txt` using the `load_config()` function:
1. Each key-value pair is parsed line by line.
2. Comments (`# ...`) and blank lines are ignored.
3. All values are validated and normalized into a `GameConfig` object.
4. If something is missing, the program applies sensible defaults (e.g., `MODE=H_VS_C`).

### Writing (`save_config`)
When you modify settings from the in-game Configuration Menu and choose to save:
1. `main.py` rewrites this file in the same `KEY=VALUE` format.
2. Optional parameters like `SEED` or `BOARD` are included only if relevant.
3. The file is overwritten atomically to prevent partial saves.

---

## Error Handling and Edge Cases

- **Missing SIZE** — The game cannot start without a board size; this raises a clear error message and prompts the user to fix it.
- **Invalid MODE** — Any unrecognized mode (e.g., a typo) is rejected and reset to a default.
- **Strategy Validation** — The selected strategies must match the chosen `MODE` (e.g., no missing strategy when a computer player is active).
- **Manual Board Validation** —  
  - If `BOARD_SOURCE=MANUAL`, the program ensures the list contains **exactly `SIZE×SIZE` integers**.  
  - Extra or missing numbers cause an explicit error.
- **Fallback to Defaults** — If optional settings (like `SEED` or `START_PLAYER`) are absent, safe defaults are applied automatically.
- **Human Safety** — The configuration UI prevents invalid combinations before writing the file, reducing the risk of corrupted settings.

---

## Example Configuration File

```txt
# Row-Column Game Configuration File
# Lines starting with '#' are comments and ignored.

SIZE=5
MODE=H_VS_C
A_STRATEGY=Greedy
B_STRATEGY=MaximizeFutureMin
BOARD_SOURCE=RANDOM
SEED=42
START_PLAYER=A

# Uncomment below to use a manual board instead of random
# BOARD_SOURCE=MANUAL
# BOARD=1,2,3,4,5,6,7,8,9,1,2,3,4,5,6,7,8,9,1,2,3,4,5,6
