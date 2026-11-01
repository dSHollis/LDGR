# Record schema

Each purchase is one JSON object on its own line in `ledger.jsonl`.

| Field | Type | Required | Notes |
|---|---|---|---|
| `ts` | string | yes | ISO 8601 timestamp with offset, e.g. `2026-10-14T18:22:05-04:00`. A date-only value (`2026-10-14`) is accepted. |
| `person` | string | yes | Who made the purchase. Lowercased on ingest. Allowed values are set in `ldgr/schema.py` (`PEOPLE`). |
| `amount` | number | yes | Positive number in dollars. A numeric string (`"42.17"`) or a `$` prefix is accepted and converted. Refunds go in as a negative amount. |
| `vendor` | string | yes | Where it was bought. Trimmed; case is kept. |
| `category` | string | yes | Lowercased. Should be one of `CATEGORIES` in `ldgr/schema.py`; unknown categories are accepted with a warning so capture never blocks. |
| `payment` | string | no | e.g. `credit`, `debit`, `cash`. Lowercased. |
| `note` | string | no | Free text. |

## Example

```json
{"ts":"2026-10-14T18:22:05-04:00","person":"scott","amount":42.17,"vendor":"Publix","category":"groceries","payment":"credit","note":"weekly shop"}
```

## Database

`ingest.py` loads records into a single `purchases` table:

| Column | Type | Notes |
|---|---|---|
| `id` | TEXT PRIMARY KEY | SHA-256 of the normalized record; makes ingest idempotent |
| `ts` | TEXT | ISO 8601 as captured |
| `date` | TEXT | `YYYY-MM-DD`, derived from `ts` in its own offset (local date of purchase) |
| `person` | TEXT | |
| `amount` | REAL | |
| `vendor` | TEXT | |
| `category` | TEXT | |
| `payment` | TEXT | nullable |
| `note` | TEXT | nullable |
| `source_line` | INTEGER | line number in the file it came from |
| `ingested_at` | TEXT | UTC timestamp of the ingest run |

Two identical purchases (same time, person, amount, vendor, category, payment and note) hash to the same id and are treated as one. In practice the timestamp to the second makes this a non-issue; if it ever matters, put something in `note`.
