# molecule-skill-cron-learnings — Per-Tick Operational Memory

Plugin for Claude Code and Hermes. End each cron tick by appending 1–3 lines of
operational learnings (what worked, what surprised, what to change next tick)
to a per-project JSONL file. Replay at the start of the next tick so the agent
carries operational context across session boundaries.

Pairs with `molecule-session-context` for auto-loading learnings at session start.

## What it does

At the end of each cron tick:
1. Review what happened — what worked, what failed, what was unexpected
2. Append 1–3 lines to `~/.claude/projects/<project>/cron-learnings.jsonl`
3. Next tick start: replay learnings so the agent knows what changed

## JSONL format

```jsonl
{"tick": "2026-04-21T14:00Z", "role": "research-lead", "learnings": ["GH_TOKEN was 401 for 3h — token refresh is fragile", "PR template needs a 'Testing' section"]}
{"tick": "2026-04-21T15:00Z", "role": "sdk-lead", "learnings": ["a2a-sdk 1.0.0 broke workspace startup — pinned to 0.3.x"]}
```

## Installation

### In org template (org.yaml)
```yaml
plugins:
  - molecule-skill-cron-learnings
```

### From URL
```
github://Molecule-AI/molecule-ai-plugin-molecule-skill-cron-learnings
```

## License
Business Source License 1.1 — © Molecule AI.
