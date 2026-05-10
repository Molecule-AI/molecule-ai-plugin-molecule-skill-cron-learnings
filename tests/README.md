# Test Rationale — molecule-skill-cron-learnings

## What this plugin does

`molecule-skill-cron-learnings` is a prose-only skill: its logic lives entirely in
`skills/cron-learnings/SKILL.md` (how to write JSONL records, storage path format,
category schema). The adapter (`adapters/claude_code.py`) is a thin re-export of
`AgentskillsAdaptor` from `plugins_registry.builtins` — no business logic, no network
calls, no side effects.

## What is tested

- `plugin.yaml` is valid YAML with required fields (name, version, runtimes, skills)
- `skills/cron-learnings/SKILL.md` has valid YAML frontmatter and a body with
  required sections (Storage, JSONL format, categories)
- `adapters/claude_code.py` exists and re-exports `AgentskillsAdaptor`
- `validate-plugin.py` (`.molecule-ci/scripts/`) exits zero

## What is NOT unit-tested (and why)

The JSONL write/replay logic is prose guidance inside SKILL.md — no Python function
to call. Testing it would require a full workspace runtime with filesystem access.
Smoke tests cover the artifact structure; full evaluation requires integration tests.

## Running tests

```bash
python -m pytest tests/ -v
```
