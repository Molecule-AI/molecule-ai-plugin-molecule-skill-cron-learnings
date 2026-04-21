# molecule-ai-plugin-molecule-skill-cron-learnings

At the end of every cron tick, append 1–3 lines of operational learnings to a
per-project JSONL file. At the start of the next tick, replay the last 20 lines
so the agent carries operational memory across session boundaries.

Pairs with `molecule-session-context` for auto-loading at session start.

**Version:** 1.0.0
**Runtime:** `claude_code`, `hermes`
**Skill:** `cron-learnings`

---

## What It Does

### End of tick — write

After every cron tick completes, append a JSON object to:
```
~/.claude/projects/<sanitized-project-path>/memory/cron-learnings.jsonl
```

```json
{"ts": "2026-04-21T14:00:00Z", "tick_id": "abc123-001", "category": "gate-fail",
 "summary": "Gate 4 (security) flagged token!=secret in PR #28",
 "next_action": "When reviewing auth-gate code, grep for subtle.ConstantTimeCompare"}
```

Categories: `gate-fail`, `mechanical-fix`, `false-positive`, `tool-error`,
`repo-state`, `pattern`.

**Rule:** 1–3 lines, be terse. `summary` = what happened; `next_action` =
what future-you should do differently. If no concrete `next_action`, don't log.

### Start of tick — replay

Before Step 1 of the cron prompt, read the last 20 lines of the JSONL (most
recent first). Use learnings to:
- Skip false-positive paths the previous tick flagged.
- Apply learned patterns.
- Avoid re-litigating decided design choices.

### Pruning

Cap at 500 lines. When exceeded, the next write drops the oldest 100 lines.
The goal is recent operational memory, not an audit log.

---

## Repository Layout

```
molecule-skill-cron-learnings/
├── skills/
│   └── cron-learnings/
│       └── SKILL.md        # Full skill definition
├── adapters/
│   ├── __init__.py
│   └── claude_code.py     # AgentskillsAdaptor (wraps plugins_registry)
├── plugin.yaml             # Plugin manifest
└── README.md
```

---

## Key Conventions

| Topic | Convention |
|---|---|
| **Format** | JSONL, one record per line |
| **Timestamp** | UTC ISO 8601 (`YYYY-MM-DDTHH:MM:SSZ`) |
| **Tick ID** | From `CRON_TICK_ID` env var if available |
| **Max per write** | 1–3 JSON lines |
| **Max file size** | 500 lines (prune oldest 100 on overflow) |
| **Character encoding** | ASCII only — no PII, tokens, or auth URLs |
| **Storage path** | `~/.claude/projects/<project>/memory/cron-learnings.jsonl` |

---

## Development

```bash
# Simulate a learning write
echo '{"ts":"2026-04-21T14:00:00Z","tick_id":"test-001","category":"pattern","summary":"test","next_action":"see above"}' \
  >> ~/.claude/projects/test/memory/cron-learnings.jsonl

# Read last 20 learnings
tail -20 ~/.claude/projects/test/memory/cron-learnings.jsonl | jq -s .

# Check file line count
wc -l ~/.claude/projects/test/memory/cron-learnings.jsonl
```

---

## Relationship to Other Plugins

- **Pairs with `molecule-session-context`** — auto-loads learnings at session start.
- **Complements `molecule-audit-trail`** — audit-trail records what changed; cron-learnings records why and what to do next.
- **Used by `molecule-workflow-triage`** — `/triage` reads learnings as part of PR review context.
