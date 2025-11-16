# main.py — File Guide

## Overview and Purpose
`main.py` functions as the **entry point and user interface layer** of the Row-Column Game.  
It manages user interaction, configuration persistence, and overall program control.  
All menu navigation, configuration editing, and game session launching originate here.  
The file bridges **user input**, **game logic** (`game_engine.py`), and **data storage** (`config.txt`, `history.csv`).

---

## Core Data Structures and Classes

### `BackCommand(Exception)`
A lightweight custom exception used to unwind nested menus when a user types `Back`.  
This keeps menu navigation structured without requiring complicated flag logic.

### UI State (dictionary)
A temporary in-memory structure used during configuration editing to preserve “raw”  
input values and track differences before saving.

**Keys**
- `board_source`: `"RANDOM"` or `"MANUAL"`  
- `seed_raw`: original seed value as string  
- `preset_board_raw`: raw string representing manual board entries  

This state allows the program to manage partially edited configurations safely.

---

## Functions

### `guarded_input(prompt: str = "") -> str`
Handles all user input consistently across menus and gameplay prompts.

- **Behavior:**  
  - Accepts user input and intercepts the global commands:  
    - `Menu` → raises `KeyboardInterrupt` to return to the main menu.  
    - `Back` → raises `BackCommand` to step up one menu layer.  
    - `Exit` → exits the program immediately.  
  - Returns the raw input otherwise.

- **Purpose:**  
  Ensures a unified interface for handling exit/navigation actions anywhere in the UI.

---

### `_parse_int_list(csv: str) -> List[int]`
Converts a comma-separated list of numbers (e.g., `"1,2,3,4"`) into a list of integers.

- **Behavior:**  
  - Strips whitespace, splits by commas, filters blanks, and converts to integers.  
  - Returns a clean list suitable for initializing `BOARD` entries in `config.txt`.  

- **Error Handling:**  
  - Raises `ValueError` if any item is not numeric.

---

### `load_config(path: str) -> GameConfig`
Reads `config.txt` and builds a `GameConfig` object.

- **Behavior:**  
  - Opens the configuration file and parses `KEY=VALUE` pairs.  
  - Applies case normalization and trims whitespace.  
  - Determines whether to load a **manual board** (`BOARD_SOURCE=MANUAL`) or a **random board** (`BOARD_SOURCE=RANDOM`).  
  - Converts numeric fields such as `SIZE` and `SEED`.  

- **Returns:**  
  A fully populated `GameConfig` instance.  

- **Raises:**  
  - `FileNotFoundError` if the config file doesn’t exist.  
  - `ValueError` for invalid or missing critical keys (`SIZE`, `MODE`).

---

### `save_config(path: str, config: GameConfig, *, board_source: str, seed_raw: str, preset_board_raw: str) -> None`
Writes the current configuration state to disk.

- **Behavior:**  
  - Opens (or creates) `config.txt`.  
  - Writes each key in `KEY=VALUE` format.  
  - Includes descriptive comment headers for readability.  
  - Omits optional fields when set to `None`.  
  - Saves manual boards as comma-separated integers.  

- **Purpose:**  
  Ensures persistent and human-readable configuration between runs.

---

### `clear_screen() -> None`
Clears the terminal display cross-platform.

- **Implementation:**  
  - Uses `os.system("cls")` on Windows.  
  - Uses `os.system("clear")` on Unix/Linux/macOS.

---

### `show_main_menu() -> None`
Renders the ASCII main menu layout.

#### Display:

```text
==============================================
       Row-Column Game (RC-Game)
==============================================

1. Play
2. Adjust Config
3. History
4. Exit

==============================================
```

- **Purpose:**  
  Provides a simple entry interface to launch a game, edit configuration, or view history.

---

### `show_history() -> None`
Displays recent match results.

- **Behavior:**  
  - Reads from `history.csv` via `scorekeeper.read_history()`.  
  - Prints the most recent entries with columns for scores, rounds, and winner.  
  - Handles empty or missing files gracefully.  

- **Purpose:**  
  Allows users to review previously played games.

---

### Configuration Submenus
These modular functions let players edit configuration interactively.

- **`submenu_board_size(config)`** — Change board dimensions (`SIZE`).  
- **`submenu_game_mode(config)`** — Toggle game mode (`H_VS_H`, `H_VS_C`, `C_VS_C`).  
- **`submenu_strategy_A(config)` / `submenu_strategy_B(config)`** — Assign strategies.  
- **`board_presets_menu(config, state)`** — Switch between random/manual board, set seed or manual board.  
- **`submenu_start_player(config)`** — Select starting player (`A` or `B`).  

Each submenu validates input, updates `config`, and supports `Back`, `Menu`, and `Exit`.

---

### `show_config_menu(config: GameConfig, state: dict) -> bool`
Runs the full configuration editing interface.

- **Behavior:**  
  - Presents the current settings.  
  - Lists numbered menu options for each parameter.  
  - Applies changes interactively through submenu calls.  
  - Returns `True` if the user chooses to save, or `False` if canceled.  

- **Purpose:**  
  Acts as the central hub for configuration customization.

---

### `main() -> None`
Controls the entire application lifecycle.

- **Sequence:**  
  1. Load or create a configuration file.  
  2. Display the main menu.  
  3. Execute user-selected options:  
     - **Play** — Launches `start_game(config)` from `game_engine.py`.  
     - **Adjust Config** — Opens `show_config_menu()`.  
     - **History** — Displays saved results via `show_history()`.  
     - **Exit** — Terminates the program.  
  4. After a game, displays post-game options (Replay / Save / Menu).  

- **Notes:**  
  The function runs continuously in a loop until `Exit` is chosen or the process is terminated.

---

## Error Handling and Edge Cases
- **Input resilience:** All menus accept `Back`, `Menu`, and `Exit` globally.  
- **Validation:** Before saving, ensures mode/strategy consistency (e.g., `C_VS_C` requires both strategies).  
- **Manual board length:** If the preset board doesn’t match the declared size, it is cleared and replaced with a random board; a warning is shown.  
- **Missing files:** `load_config()` creates new defaults if `config.txt` is absent.  
- **History viewer:** Catches `FileNotFoundError` or malformed CSV gracefully and displays an empty history instead of crashing.  
- **Display safety:** Clears the screen at each major stage to prevent text clutter during user navigation.

---
