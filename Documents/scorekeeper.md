# scorekeeper.py — File Guide

## Overview and Purpose
`scorekeeper.py` provides the **persistence layer** for recording and retrieving match results.  
It maintains the `history.csv` file, where each completed game’s details are logged, including scores, mode, and winner.  
The file can later be read and displayed by the History viewer in `main.py`.

---

## Core Data Structures and Classes

### `HISTORY_FILENAME`
- **Type:** `str`  
- **Value:** `"history.csv"`  
- **Purpose:** The persistent file used to store all game outcomes.

### `FIELDNAMES`
- **Type:** `List[str]`  
- **Value:**  
  ```python
  ["date", "match", "a_score", "b_score", "winner", "rounds", "mode"]
  ```  
- **Purpose:** Defines the order and names of the columns in `history.csv`.  
  Ensures consistent field alignment for both writing and reading.

---

## Functions

### `_history_path() -> str`
Determines the absolute path to `history.csv`.

- **Behavior:**  
  - Resolves a file path adjacent to the script directory, ensuring portability.  
  - Centralizes the path logic to prevent repetition across functions.  

- **Returns:**  
  - A valid string path to the history file.

---

### `_ensure_file_exists() -> None`
Guarantees that the `history.csv` file exists and contains a valid header row.

- **Behavior:**  
  - Checks for the presence of the file.  
  - If missing, creates it and writes the CSV header (`FIELDNAMES`).  

- **Purpose:**  
  - Prevents file-not-found errors when appending or reading history.  

---

### `append_result(match: int, a_score: int, b_score: int, winner: str, rounds: int, mode: str) -> None`
Appends a new result row to the `history.csv` file.

- **Behavior:**  
  - Ensures the file exists via `_ensure_file_exists()`.  
  - Opens the CSV in append mode (`'a'`).  
  - Adds a row with:  
    - Timestamp (`date`)  
    - Match number (`match`)  
    - Player A score (`a_score`)  
    - Player B score (`b_score`)  
    - Winner (`winner`)  
    - Rounds played (`rounds`)  
    - Game mode (`mode`)  

- **Example Output Row (CSV):**
  ```csv
  date,match,a_score,b_score,winner,rounds,mode
  2025-11-16 15:42,1,42,38,A,21,H_VS_C
  ```

- **Purpose:**  
  Maintains a cumulative history of all sessions for later review or analysis.

---

### `read_history() -> List[Dict[str, str]]`
Reads the contents of `history.csv` and returns them as a list of dictionaries.

- **Behavior:**  
  - Ensures the file exists before attempting to read.  
  - Parses each row using the field names.  
  - Ignores any unexpected extra columns gracefully.  

- **Returns:**  
  - A list of dictionaries, one per recorded game.  
  - Returns an empty list (`[]`) if the file is empty or missing.  

- **Example Output:**
  ```python
  [
      {
          "date": "2025-11-16 15:42",
          "match": "1",
          "a_score": "42",
          "b_score": "38",
          "winner": "A",
          "rounds": "21",
          "mode": "H_VS_C"
      }
  ]
  ```

---

## Error Handling and Edge Cases
- **Automatic file creation:** `_ensure_file_exists()` guarantees a valid CSV before any operation.  
- **Safe writing:** All values are stringified to preserve formatting and prevent type mismatches.  
- **Missing file:** If no file exists, reading returns an empty list rather than raising an error.  
- **Malformed data:** Extra or unknown columns are ignored, ensuring backward compatibility with older file versions.  
- **Concurrency safety:** Files are opened and closed atomically; no persistent handles are kept between writes.

---
