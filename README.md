# Grid Monitoring Prototype

A small Python + SQLite console application that simulates grid load 
monitoring: it takes load readings as input, calculates a simple 
risk score, flags overload conditions, and logs each reading to a 
local database.

## What it does

- Prompts for a load value on each cycle
- Calculates a risk score from load and a fixed temperature value
- Flags readings above a configurable threshold as overload risk
- Persists every reading (timestamp, load, risk score) to SQLite

## Why I built it

Self-directed project to build foundational Python and database 
skills (control flow, functions, SQLite I/O, input validation) 
alongside my EEE coursework. Not affiliated with any research 
programme or infrastructure body — a personal learning project.

## Status

Early / ongoing. Current version is a single-file console script. 
Planned next steps: input validation cleanup, config file for 
thresholds, basic unit tests.

## Usage

\`\`\`bash
python grid_monitor.py
\`\`\`

Enter a numeric load value when prompted, or type `exit` to quit.

import sqlite3
from datetime import datetime

DB_PATH = "grid_telemetry.db"
RISK_THRESHOLD = 85.0
LOAD_WEIGHT = 1.2


def get_connection(db_path: str = DB_PATH) -> sqlite3.Connection:
    conn = sqlite3.connect(db_path)
    conn.execute("""
        CREATE TABLE IF NOT EXISTS system_logs (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
            load_value REAL NOT NULL,
            risk_value REAL NOT NULL
        )
    """)
    conn.commit()
    return conn


def evaluate_grid_stability(load_percent: float, temperature: float) -> float:
    """Compute a simple risk score from load and ambient temperature."""
    return (load_percent * LOAD_WEIGHT) + temperature


def log_reading(conn: sqlite3.Connection, load_value: float, risk_value: float) -> None:
    conn.execute(
        "INSERT INTO system_logs (load_value, risk_value) VALUES (?, ?)",
        (load_value, risk_value),
    )
    conn.commit()


def parse_load_input(raw: str) -> float | None:
    try:
        return float(raw)
    except ValueError:
        print(f"Invalid input: '{raw}' is not a number.")
        return None


def run_monitoring_loop(conn: sqlite3.Connection) -> None:
    while True:
        raw_input_value = input("\nEnter load input (or 'exit'): ").strip()

        if raw_input_value.lower() == "exit":
            print("Shutting down monitoring session.")
            break

        load_value = parse_load_input(raw_input_value)
        if load_value is None:
            continue

        risk_score = evaluate_grid_stability(load_value, temperature=25.0)
        status = "CRITICAL OVERLOAD" if risk_score > RISK_THRESHOLD else "NOMINAL"
        print(f"Status: {status} (risk={risk_score:.2f})")

        log_reading(conn, load_value, risk_score)


def main() -> None:
    conn = get_connection()
    try:
        run_monitoring_loop(conn)
    finally:
        conn.close()
        print("Database connection closed.")


if __name__ == "__main__":
    main()

