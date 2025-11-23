# Row-Column Game (RC-Game)

## Overview
The **Row-Column Game (RC-Game)** is a terminal-based, turn-based strategy game written entirely in Python.  
Two players—either human or computer—take turns selecting numbered cells from a square grid. Once a cell is taken, it is removed from the board.  
Every move after the first must be chosen from the **same row or column** as the opponent’s last move.  
The player with the higher total score at the end of the game wins.

The project serves as both a playable game and a demonstration of **clean modular architecture**, **algorithmic game strategizing**, and **structured Python design**.

---

## Key Features
- Pure **Python 3.7+** implementation (no third-party dependencies)
- Interactive command-line interface
- Multiple play modes: Human vs Human, Human vs Computer, Computer vs Computer
- Pluggable AI strategies (`Greedy`, `MaximizeFutureMin`, `MinimizeOpponentOptions`, `PreserveHighValues`)
- Persistent configuration via `config.txt`
- Automatic match logging in `history.csv`
- Modular design allowing easy extension and testing

---

## Repository Structure
```
RC-Game/
│
├── LICENSE
├── README
│
├── Game_Code/
│   ├── .gitkeep
│   ├── LICENSE
│   ├── board_manager.py
│   ├── config.txt
│   ├── game_engine.py
│   ├── history.csv
│   ├── main.py
│   ├── requirements.txt
│   ├── scorekeeper.py
│   └── strategies.py
│
└── Documents/
    ├── ARCHITECURE_GUIDE.md
    ├── USER_MANUAL_FINAL.md
    ├── board_manager.md
    ├── config.md
    ├── game_engine.md
    ├── main.md
    ├── scorekeeper.md
    └── strategies.md
```

---

## Installation and Setup

### Requirements
- Python **3.7** or newer  
- Works on Windows, macOS, and Linux (console required)

### Steps
1. Clone or download the repository:
   ```bash
   git clone https://github.com/<your-username>/row-column-game.git
   cd row-column-game
   ```
2. Run the game:
   ```bash
   python main.py
   # or
   python3 main.py
   ```

No additional dependencies are required.

---

## Gameplay Summary

### Objective
Collect the highest total score by selecting numbers from the grid while respecting the **row/column constraint**.

### Basic Rules
1. Player A moves first.  
   - If human, they may choose any cell.  
   - If computer-controlled, the assigned AI strategy selects from all available cells.
2. All subsequent moves must be chosen from the **same row or column** as the opponent’s previous move.
3. Each chosen number is added to the current player’s score.
4. The game ends when no valid moves remain. The player with the higher score wins.

### Example Round
```
Round 3 (5x5)
┌───────────────┐
│ 1  3  5  2  9 │
│ 4  7  X  1  6 │
│ 5  2     8  4 │
│ 3  9  6  7  1 │
│ 8  4  5  9  2 │
└───────────────┘
Score A: 8 | Score B: 7
```

---

## Configuration

Game settings are defined in `config.txt`.  
You can edit this file manually or use the in-game **Configuration Menu**.

### Example Configuration
```
SIZE=5
MODE=H_VS_C
START_PLAYER=A
BOARD_SOURCE=RANDOM
A_STRATEGY=Greedy
B_STRATEGY=MaximizeFutureMin
SEED=42
```

### Configuration Fields
| Field | Description |
|--------|-------------|
| `SIZE` | Board size (`N×N`, must be ≥2) |
| `MODE` | Game mode (`H_VS_H`, `H_VS_C`, `C_VS_C`) |
| `A_STRATEGY` | Strategy for Player A (if computer-controlled) |
| `B_STRATEGY` | Strategy for Player B (if computer-controlled) |
| `START_PLAYER` | Starting player (`A` or `B`) |
| `BOARD_SOURCE` | `RANDOM` or `MANUAL` |
| `SEED` | Optional RNG seed for reproducible boards |
| `BOARD` | Manual board values (comma-separated, optional) |

---

## Available Computer Strategies

| Strategy | Description |
|-----------|-------------|
| **Greedy** | Selects the highest available cell value. |
| **MaximizeFutureMin** | Simulates one step ahead, maximizing the player’s immediate score while minimizing the opponent’s next possible gain. |
| **MinimizeOpponentOptions** | Chooses moves that minimize the opponent’s future available positions. |
| **PreserveHighValues** | Avoids exposing high-value cells to the opponent; prioritizes board control and defense. |

Each strategy is independent and interchangeable.  

---

## Match History

Each completed game is logged in `history.csv` for later review.  
A new file is automatically created on first use.

### Example Entry
```
date,match,a_score,b_score,winner,rounds,mode
2025-11-16 15:32,Alice VS Bob,42,38,A,21,H_VS_C
```

You can view the most recent results through the **History** option in the main menu.
```
================================================================================
                 History
================================================================================

Showing up to the last 20 results:

 1. 2025-11-15 20:09 | BOB VS LEO | A:24 B:14 | Winner:A | Rounds:7 | Mode:C_VS_C
 2. 2025-11-15 20:10 | CHARLIE VS SON | A:27 B:22 | Winner:A | Rounds:8 | Mode:C_VS_C
```
---

## Design Overview

The project emphasizes modularity and maintainability.  
Each module handles a distinct layer of functionality:

```
┌──────────────────────────────┐
│          main.py             │
│  • User Interface            │
│  • Config & Menu Handling    │
└────────────┬─────────────────┘
             │
             ▼
┌──────────────────────────────┐
│        game_engine.py        │
│  • Game Loop & Flow Control  │
│  • Turn Validation & Scoring │
└────────────┬─────────────────┘
             │
     ┌───────▼────────┐   ┌───────────────┐
     │ board_manager  │   │  strategies   │
     │ • Board Logic  │   │ • Behavior    │
     │ • Display      │   └───────────────┘
     └────────────────┘
             │
             ▼
┌────────────────────────────┐
│       scorekeeper.py       │
│     • History Logging      │
└────────────────────────────┘
```

Further technical details are available in **ARCHITECTURE_GUIDE.md**.

---

## Documentation

| File | Purpose |
|------|----------|
| `USER_MANUAL_FINAL.md` | Detailed player guide, setup, and strategy explanations. |
| `ARCHITECTURE_GUIDE.md` | Full technical documentation of modules, flow, and design. |
| `docs/*.md` | Per-file documentation for each Python module. |

---

## Design Principles
- **Single Responsibility:** Each module serves one clear purpose.  
- **Loose Coupling:** Minimal interdependence between components.  
- **Transparency:** Game state and logic are printed clearly each round.  
- **Extensibility:** New strategies or game rules can be added easily.  
- **Reproducibility:** Seeding allows identical boards for testing and benchmarking.

---

## License
This project is distributed under the **MIT License**.  
You may use, modify, and distribute it freely for educational or personal use.

---

## Contacts
**Author(s):** Rémi FERRARI, Sandro Weeks, Fabrizio Gorelli, Diego Lafontaine, Gent Jusufi, Sal Vassallo

---
