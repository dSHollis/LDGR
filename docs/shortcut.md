# Siri Shortcut capture

The Shortcut asks a few questions, builds one JSON line, and appends it to `ledger.jsonl` in iCloud Drive (or any folder the ingest machine can reach).

## Build it

In the Shortcuts app, create a new Shortcut named **Log Purchase** with these actions:

1. **Ask for Input** — *Number*, prompt `Amount?` → variable `Amount`
2. **Ask for Input** — *Text*, prompt `Vendor?` → `Vendor`
3. **Choose from List** — items: `groceries`, `dining`, `fuel`, `household`, `health`, `entertainment`, `travel`, `gifts`, `subscriptions`, `other`; prompt `Category?` → `Category`
4. **Choose from List** — items: `credit`, `debit`, `cash`; prompt `Paid with?` → `Payment`
5. **Ask for Input** — *Text*, prompt `Note? (optional)`, allow empty → `Note`
6. **Format Date** — *Current Date*, format **ISO 8601**, include time → `TS`
7. **Dictionary** with keys:
   - `ts` → `TS`
   - `person` → `scott` (hard-code per phone; your partner's copy uses their name)
   - `amount` → `Amount` (set the value type to *Number*)
   - `vendor` → `Vendor`
   - `category` → `Category`
   - `payment` → `Payment`
   - `note` → `Note`
8. **Get Text from Input** on the Dictionary (produces compact JSON)
9. **Append to Text File** — file `iCloud Drive/LDGR/ledger.jsonl`, **Make New Line** on

Add it to the Home Screen or Action Button, and "Hey Siri, log purchase" works too.

## Tips

- Keep the category list identical to `CATEGORIES` in `ldgr/schema.py`. Unknown categories still ingest, but with a warning.
- Each phone hard-codes its own `person`, which removes one prompt and one source of typos.
- If both phones append to the same iCloud file at the same moment, iCloud can create a conflicted copy. Two separate files (`ledger-scott.jsonl`, `ledger-partner.jsonl`) avoid that entirely; ingest accepts multiple files.
- Test by running the Shortcut once, then `tail -n 1 ledger.jsonl` on the ingest machine and check it parses with `python -m json.tool`.
