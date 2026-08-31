# Platform Routing

Read this reference immediately before delegating. Use the native primitives actually available in the current runtime; do not fabricate a cross-platform command or assume an experimental feature is enabled.

## Shared abstraction

Map the operating model to three roles:

- **Lead:** owns the task charter, authority boundary, work ledger, integration, and user communication.
- **Worker:** owns one bounded workstream and returns evidence plus artifacts.
- **Reviewer:** independently evaluates the integrated result against acceptance criteria.

Prefer in-session workers for bounded subtasks. Prefer isolated sessions or worktrees when concurrent writes may collide, the task is long-running, or the user needs to inspect each session directly.

## OpenAI Codex

- Use the collaboration or subagent tools exposed by the current Codex surface for internal workstreams. Common operations are spawn, message or follow up, list, wait, and interrupt.
- Use separate user-visible Codex tasks or threads only when the user explicitly asks to create or hand off a task. Do not create sidebar tasks merely to implement an internal subtask.
- Assume in-session agents may share the same filesystem unless the runtime explicitly provides isolation. Assign exclusive write ownership, keep other agents read-only, or use isolated worktrees when available.
- Fork only the context a worker needs. Give full history when prior discussion is essential; otherwise send a self-contained task contract to reduce noise and accidental scope inheritance.
- Wait on active work with the runtime's coordination primitive and steer the same worker with follow-ups. Avoid duplicate agents for the same task unless intentionally testing competing hypotheses.

If native subagents are unavailable, execute the workstreams sequentially and retain the same contracts and verification gates.

## Claude Code

- Use subagents for side tasks within one session, especially high-volume exploration that should return a concise result.
- Use agent view or separate worktree sessions when several independent tasks should remain directly inspectable by the user.
- Use agent teams only when workers need a shared task list or direct inter-agent communication. Treat teams as optional because availability and enablement can vary.
- For concurrent code changes, prefer a worktree-isolated subagent or partition exclusive file ownership. Agent teams do not inherently prevent write collisions.
- Use a named reusable subagent definition when the same specialist role recurs; ordinary one-off work can use a focused task contract without creating permanent configuration.
- A Claude skill can be loaded by a subagent when supported, but the lead must still pass task-specific scope, acceptance criteria, and authority boundaries.

If the relevant mode is unavailable, fall back to ordinary subagents or sequential work rather than changing the user's configuration without permission.

## Local, worktree, or remote isolation

Choose isolation based on the bottleneck and verification needs rather than vendor preference:

- Use a local in-session worker for quick discovery, read-only investigation, or work that depends on the lead's exact environment.
- Use a worktree or equivalent isolated checkout for concurrent code changes. Remember that separate filesystems can still collide through shared ports, databases, caches, test accounts, or external services.
- Use a remote or cloud worker when the task benefits from a separate machine, sustained background execution, or greater concurrency, and the environment can be bootstrapped reproducibly. Require it to return the same verification artifacts as a local worker.

Prove one end-to-end path in the target environment before fanning out. Do not change remote-agent configuration, secrets, or paid capacity without the authority that would be required for the lead to make the same change.

## Model routing

Do not hard-code a vendor model name into a durable workflow unless the user requests it. Capabilities and prices change.

- Use a strong model for ambiguous decomposition, architecture, synthesis, and consequential review.
- Use a faster or cheaper model for bounded retrieval, formatting, enumeration, or deterministic checks when measured quality remains acceptable.
- Use different models for author and judge when this provides meaningful error diversity, but do not treat model disagreement as evidence by itself.
- Preserve a user-selected model and organization allowlist. Never silently substitute a preferred vendor or model family.

## Cross-platform handoff

When Codex and Claude both participate, exchange artifacts rather than hidden conversational state:

```markdown
Task charter and current decision:
Owned scope:
Inputs and exact paths:
Changes or findings:
Verification evidence:
Open risks and next action:
```

Keep one integration owner. Do not let both platforms edit overlapping files concurrently unless they work in isolated branches with an explicit reconciliation step.
