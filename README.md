# memory-governance

`memory-governance` is an agent-side safety skill for memory operations.

It helps agents:
- decide when short-term notes can be promoted to long-term memory or knowledge,
- enforce consent/privacy rules by scope (`private` vs `shared`),
- detect contradictions before updates (via `rg`/`grep`) and ask users to resolve.

## Install

Copy this folder into your local skills directory.

### OpenClaw

Place it under your OpenClaw skills workspace, for example:

```bash
cp -R memory-governance /path/to/openclaw/skills/
```

### Claude Code

Place it under:

```bash
~/.claude/skills/memory-governance
```

### Codex

Place it under:

```bash
~/.agents/skills/memory-governance
```

## Usage

Invoke the skill when handling memory save/update/delete flows, especially when:
- scope is unclear,
- long-term or knowledge updates may conflict with existing entries,
- shared context requires explicit permission checks.
