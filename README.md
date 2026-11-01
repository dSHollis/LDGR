# LDGR

A small, two-person purchase-logging pipeline. Purchases are captured in a few seconds from an iOS (Siri Shortcut), written as one JSON line each to an append-only text file, ingested into SQLite, and analyzed from the command line.

```
 Siri Shortcut 
            │  (a few prompts per purchase)
            ▼
   ledger.jsonl   ← append-only raw record, one JSON object per line
            │
            ▼  ldgr/ingest.py   (validate, dedupe, load)
   ldgr.db  (SQLite)
            │
            ▼  ldgr/analyze.py  (flags → tables and plots)
   summaries · charts
```

## Why it's built this way

- Capture has to be effortless. A logging system nobody feeds is useless, so the input side is a single tap and a few prompts.
- The raw file is the source of truth. `ledger.jsonl` is never edited by the pipeline. The database can be deleted and rebuilt from it at any time.
- Each line is hashed. Re-running ingest on the same file never creates duplicates.
- Ingest and analysis are separate. Loading data and asking questions of it are different jobs with different failure modes.

## Record format

One JSON object per line. Full details in [`docs/schema.md`](docs/schema.md).

```json
{"ts":"2026-10-14T18:22:05-04:00","person":"scott","amount":42.17,"vendor":"Publix","category":"groceries","payment":"credit","note":""}
```

## Tests

```bash
python -m unittest discover -s tests -v
```

## Ingest

```bash
python -m ldgr.ingest PATH/TO/ledger.jsonl [--db ldgr.db] [--strict]
```

- Valid lines are inserted; already-seen lines are skipped.
- Invalid lines are reported with their line number and reason, and are written to `rejects.jsonl` next to the database so nothing is silently lost.
- `--strict` exits non-zero if any line is rejected (useful in a cron job).

## Analyze

```bash
python -m ldgr.analyze --db ldgr.db [options]
```

| Flag | Meaning |
|---|---|
| `--by {category,person,vendor,payment,month,week,day}` | Group totals by this field (default `category`) |
| `--person NAME` | Filter to one person (`both` = no filter, default) |
| `--category NAME` | Filter to one category |
| `--since YYYY-MM-DD` / `--until YYYY-MM-DD` | Date range (inclusive) |
| `--top N` | Show only the top N groups |
| `--plot` | Draw a chart of the grouped totals |
| `--out FILE.png` | Save the chart instead of opening a window |
| `--csv FILE.csv` | Export the grouped table |
| `--split` | With `--by month/week/day`, plot one line per person |

Examples:

```bash
# Where did October go?
python -m ldgr.analyze --since 2026-10-01 --until 2026-10-31 --by category

# Monthly spending per person, saved as an image
python -m ldgr.analyze --by month --split --plot --out monthly.png

# Top 10 vendors this year
python -m ldgr.analyze --since 2026-01-01 --by vendor --top 10
```

## Capture setup

See [`docs/shortcut.md`](docs/shortcut.md) for building the Siri Shortcut.

## Repo layout

```
ldgr/
├── ldgr/
│   ├── __init__.py
│   ├── schema.py      # field definitions and validation
│   ├── db.py          # SQLite connection and table setup
│   ├── ingest.py      # JSONL → SQLite
│   └── analyze.py     # queries, tables, plots
├── docs/
│   ├── schema.md
│   └── shortcut.md
├── examples/
│   ├── make_sample.py        # regenerates the fake data
│   └── sample_ledger.jsonl   # fake data only
├── tests/
│   └── test_pipeline.py
├── LICENSE
├── requirements.txt
└── .gitignore
```

## Privacy

Real purchase data never goes in this repo. `.gitignore` excludes `*.db`, `ledger*.jsonl` and `rejects.jsonl`; only `examples/sample_ledger.jsonl`, which is generated fake data, is tracked.

## Status

Personal project, rebuilt and documented in 2026. Works for its intended scale (two people, thousands of records); not designed as a multi-user app.

## License

MIT
