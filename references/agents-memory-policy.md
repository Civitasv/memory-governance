## Memory Governance Policy (Global)

- Default Agent description file: OpenClaw uses `AGENTS.md`, Claude Code uses `CLAUDE.md`, Codex defaults to `AGENTS.md`.
- Discover long-term index target, daily-notes path, and knowledge target (if defined) from the Agent description file before any memory action.
- Do not assume fixed paths such as `MEMORY.md` or `memory/YYYY-MM-DD.md` unless explicitly defined.
- If the Agent description file is missing, or memory targets are not defined: pause memory actions and ask user to provide/confirm paths.
- Determine `context_scope` first:
- `private`: memory read/write allowed by policy.
- `shared`: long-term memory read is blocked; daily-documents/knowledge access requires user confirmation before any read/write.
- If scope is unknown, pause and ask user to confirm scope, and explicitly explain `private/shared` permission differences.
- Always write short-term session facts to the discovered daily-notes target.
- Promote short-term memory to long-term index only with explicit user consent (for example: "remember this", "save to long-term memory").
- Promote short-term memory to knowledge store only when content is reusable/generalizable and a knowledge target is defined.
- Use promotion criteria before promotion: Stability, Reuse, Specificity, Sensitivity, Scope fit.
- If criteria are weak, keep content in daily documents and do not promote.
- If a new preference conflicts with an existing long-term preference, ask for confirmation before overwrite.
- Before updating long-term index or knowledge, search target docs using `rg`/`grep` for related/contradictory entries.
- If contradictory entries are found, pause and ask user to confirm how to resolve (`replace / merge / keep existing`).
- For "forget/delete" requests, first locate the exact target (prefer `id` or section key), then delete and return "deleted summary + file path".
- If memory is cited in `private` scope, append: `Memory reference: <file>#<section>`; do not expose local file paths in `shared` scope.
