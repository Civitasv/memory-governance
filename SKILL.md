---
name: memory-governance
description: Use when deciding whether short-term session memory should be promoted into long-term memory or a knowledge store, and writes must stay auditable, conflict-safe, and privacy-scoped.
license: MIT
compatibility: Works in OpenClaw, Claude Code, and Codex style environments; requires local file read/write and shell search tools (rg/grep) for contradiction checks.
metadata:
  author: civitasv
  version: "0.1.0"
  spec: agentskills-v1
  tags:
    - memory
    - governance
    - privacy
    - knowledge-management
    - openclaw
    - claude-code
    - codex
---

# Memory Governance

Agent-side write guardrails for safe, auditable memory operations across platforms.

## Memory model (four layers)

- Short-term memory: current session context.
- Daily documents: raw notes and logs, not fully structured.
- Long-term memory (shallow layer): lightweight summaries of stable knowledge, preferences, and decisions.
- Knowledge store: structured and reusable knowledge.

## Discovery from Agent description file

Use this platform mapping to locate the Agent description file:

- OpenClaw default Agent description file: `AGENTS.md`
- Claude Code default Agent description file: `CLAUDE.md`
- Codex default Agent description file: `AGENTS.md`

After locating the Agent description file, discover memory targets from its content:

- Long-term memory index target (file or directory)
- Daily notes path pattern
- Knowledge-store target (file or directory), if defined

Do not assume fixed paths like `MEMORY.md` or `memory/YYYY-MM-DD.md` unless the Agent description file explicitly defines them.

If the Agent description file is missing, or memory targets are not defined:

- Pause memory read/write/delete/update actions.
- Ask the user to confirm paths for:
  - long-term index location
  - daily documents location
  - knowledge-store location (if used)
- If paths are still missing after confirmation, ask whether to create defaults:
  - long-term memory file: `MEMORY.md`
  - daily notes directory: `memory/daily/`
  - knowledge directory: `memory/knowledge/`
- If user agrees to create targets, also append/update a `## Memory` block in the Agent description file using the user-confirmed paths.
- Resume only after user confirmation.

## Scope policy (privacy first)

Decide `context_scope` before any memory action:

- `private`: direct one-on-one session; memory read/write allowed by policy.
- `shared`: group/external/multi-party context:
  - long-term memory: reading is blocked.
  - daily documents and knowledge store: ask user first before any read/write operation.
- If scope is unknown, ask user to confirm scope and explain permissions:
  - `private`: memory read/write allowed by policy.
  - `shared`: long-term memory read is blocked; daily/knowledge access requires user confirmation.

## Promotion policy

- Always persist short-term session facts to the discovered daily-notes path.
- Promote short-term/daily -> long-term index only with explicit user consent (examples: "remember this", "save to long-term memory").
- Promote short-term/daily -> knowledge store when content is reusable/generalizable and the knowledge target is defined.
- For conflicting long-term preferences, ask for confirmation before overwrite.
- Before updating long-term index or knowledge, use `rg`/`grep` to search target documents for existing related or contradictory entries.
- If contradictory content is detected, alert the user and request confirmation before applying changes.

## Promotion criteria (agent decision aid)

Use these checks before promoting short-term content:

- Stability: likely to remain valid for at least 30 days.
- Reuse: likely useful in future sessions or tasks.
- Specificity: specific enough to be actionable.
- Sensitivity: safe to persist under current privacy constraints.
- Scope fit: appropriate for target (`long-term index` vs `knowledge store`).

If checks are weak, keep content in daily documents and do not promote.

## Operational checklist

1. Classify request: `save / update / delete / read-only`.
2. Determine `context_scope`: `private / shared` (if unknown, ask user and pause).
3. In `shared` scope, enforce access gate: block long-term memory reads; ask user before daily/knowledge access.
4. Resolve Agent description file by platform defaults.
5. Discover long-term index target, daily-notes path, and knowledge target from that file.
6. If targets are missing, ask user for path confirmation; if still missing, ask whether to create default targets and pause.
7. If user confirms creation, create targets and update Agent description file `## Memory` section with chosen paths.
8. Classify memory intent: `short-term only / promote to long-term-index / promote to knowledge`.
9. For long-term/knowledge updates, run `rg`/`grep` on target docs to detect related or contradictory entries.
10. Choose target memory file/path by policy.
11. Make minimal file edit.
12. Send standard receipt message.
13. Apply citation policy for normal replies.

## Decision table

| Condition                                                   | Action                                                                                   |
| ----------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `short-term session fact`                                   | Write discovered daily-notes target                                                      |
| `promote to long-term-index + private + explicit consent`   | Write discovered long-term index target                                                  |
| `promote to long-term-index + no explicit consent`          | Do not write long-term index; ask for consent                                            |
| `promote to long-term-index + criteria weak`                | Keep in daily documents only                                                             |
| `promote to knowledge + target defined`                     | Write discovered knowledge target                                                        |
| `promote to knowledge + criteria weak`                      | Keep in daily documents only                                                             |
| `promote to knowledge + target missing`                     | No-op: do not perform knowledge write                                                    |
| `update long-term index/knowledge + contradiction detected` | Alert user with contradiction summary and ask for confirmation before update             |
| `shared context + long-term memory read`                    | Block operation                                                                          |
| `shared context + daily/knowledge access requested`         | Ask user for permission before access                                                    |
| `unknown context_scope`                                     | Ask user to confirm scope and explain `private/shared` permissions; pause memory actions |
| `missing Agent description file`                            | Ask user to provide/confirm Agent description file path; pause memory actions            |
| `missing memory targets in Agent description file`          | Ask user to provide/confirm target paths; if unresolved, ask to create default targets   |
| `user approved target creation`                             | Create targets and insert/update `## Memory` section in Agent description file           |
| `conflicting preference update`                             | Ask before overwrite                                                                     |

## Delete/update policy

- On "forget/delete a memory", first locate the exact target (prefer stable `id` or exact line/section key), then delete.
- Confirm completion with short deleted summary and target type; include full file path only if user explicitly requests it.
- On "update preference", replace only the smallest relevant block.
- On long-term index/knowledge update, first search target docs with `rg`/`grep`; if new content contradicts existing entries, show both sides and ask user whether to replace, merge, or keep existing.

## Citation and disclosure policy

- If a reply relies on stored memory in `private` scope, append: `Memory reference: <memory-id-or-source-key>`.
- If scope is `shared`, do not expose local file paths in the reply.
- If no memory is used, do not add citation line.

## Receipt templates

- Write success: `Written: <target-type> updated (id: <id-or-key>).`
- Delete success: `Deleted: <summary> (target: <target-type>, id: <id-or-key>).`
- Path request (missing Agent description file): `Memory action paused: Agent description file not found. Please provide or confirm the Agent description file path.`
- Path request (missing memory targets): `Memory action paused: long-term index / daily documents / knowledge targets are not fully defined. Please provide or confirm paths.`
- Create request (missing targets): `Memory targets are still missing. Create defaults? long-term file: MEMORY.md, daily directory: memory/daily/, knowledge directory: memory/knowledge/`
- Create success: `Created memory targets and updated Agent description file Memory section with chosen paths.`
- No-op (missing knowledge target): `No memory action taken: knowledge target is not defined in Agent description file.`
- Scope request (unknown scope): `Memory action paused: context scope is unclear. Please confirm scope. private = memory read/write allowed by policy; shared = long-term memory read blocked, and daily/knowledge access requires confirmation.`
- Permission request (shared daily/knowledge): `Shared scope detected. Please confirm permission to access daily documents/knowledge for this task.`
- Contradiction request: `Update paused: contradictory memory/knowledge detected. Existing: <A>. New: <B>. Please confirm: replace / merge / keep existing.`

## Common mistakes

- Writing to assumed default files without reading Agent description file first.
- Updating long-term memory/knowledge without searching existing entries first (`rg`/`grep`).
- Writing long-term index without explicit user intent.
- Promoting short-term memory to long-term index or knowledge without intent classification.
- Reading long-term memory in shared contexts.
- Accessing daily documents or knowledge in shared contexts without user confirmation.
- Overwriting full sections when only one preference changed.
- Returning "done" without a clear target/id and change summary.

## AGENTS.md integration

When user asks to enforce globally, add policy from `references/agents-memory-policy.md` into the platform Agent description file (`AGENTS.md` or `CLAUDE.md`) with minimal edits.

## Memory section template (for creation flow)

When user approves creating targets, write/update this in Agent description file with user-selected paths:

```markdown
## Memory

You wake up fresh each session. These files are your continuity:

- **Daily notes:** `<DAILY_DIR>/YYYY-MM-DD.md` (create `memory/` if needed) — raw logs of what happened
- **Long-term:** `<LONG_TERM_FILE>` — your curated memories, like a human's long-term memory
- **Knowledge:** `<KNOWLEDGE_DIR>/<KNOWLEDGE>-YYYY-MM-DD`, your knowledge
```
