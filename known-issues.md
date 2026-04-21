# Known Issues

Active and recently resolved issues for `molecule-skill-cron-learnings`.

---

## Active Issues

*(None currently open. File an issue if you encounter a problem.)*

---

## Known Gotchas

### Tick ID may not be available in all runtimes

**Severity:** Low
**Detail:** `tick_id` in each record uses `CRON_TICK_ID` from the environment. This env var is set by the Hermes cron scheduler but may not be present in all runtime contexts. Records without a `tick_id` are still valid — the field is optional.

---

### JSONL is not validated before write

**Severity:** Low
**Impact:** Malformed JSON in the JSONL file (e.g., from a buggy append) can corrupt the file for the next reader. Each append writes a single well-formed JSON line — as long as the skill implementation is correct, corruption is not possible. However, if another process (e.g., a manual edit) corrupts the file, the next read may fail.

**Workaround:** If the JSONL is corrupted, truncate to the last valid line:

```bash
# Find the last valid JSON line
python3 -c "
import json, sys
with open('cron-learnings.jsonl') as f:
    lines = f.readlines()
valid = []
for line in lines:
    try:
        json.loads(line.strip())
        valid.append(line)
    except:
        break
with open('cron-learnings.jsonl', 'w') as f:
    f.writelines(valid)
"
```

---

### No dedup — same mistake logged across ticks results in duplicate entries

**Severity:** Informational
**Detail:** If an agent makes the same mistake in consecutive ticks, the same lesson may appear in multiple records. This is intentional — dedup would lose temporal ordering. Use `tail -20 | sort -u` when reading if dedup is desired.

---

### `.claude/projects/` path sanitization is runtime-dependent

**Severity:** Informational
**Detail:** The sanitized project path (used in the storage path) may differ between runtimes. Hermes uses `-` separators; Claude Code uses `/`. This means the same project on the same machine may have different storage paths depending on which runtime writes first.

---

## Recently Resolved

*(None yet.)*
