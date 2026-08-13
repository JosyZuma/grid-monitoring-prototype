# Grid Monitoring Prototype

A small Python + SQLite console application that simulates grid load 
monitoring: it takes load readings as input, calculates a simple 
risk score, flags overload conditions and logs each reading to a 
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

Enter a numeric load value when prompted or type `exit` to quit.



