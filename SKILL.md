---
name: product-agent-orchestrator
description: Create and manage a coordinated multi-agent Codex product team using a product-chain collaboration model. Use when the user explicitly asks to build a product with multiple agent conversations, create role-specific Codex threads, coordinate CEO/Product Owner, design, development, and QA agents, manage cross-session shared state, define agent roles, enable direct Designer-Developer and Developer-QA collaboration, or turn a product idea into a managed multi-agent workflow.
---

# Product Agent Orchestrator

Use this skill to turn a product idea into a managed set of Codex conversations with explicit roles, shared state, task ownership, and handoff protocol.

Do not create separate Codex threads unless the user explicitly asks for multi-agent/session creation or invokes this skill for that purpose. If the user only asks for product advice, create a plan or repo-local state without opening new threads.

## Default Product Collaboration Pattern

For product development, default to this chain:

```text
User <-> CEO / Product Owner <-> Product Designer <-> Product Developer <-> QA Verifier
```

The CEO / Product Owner primarily talks with the user, clarifies ideas, decides priorities, accepts scope changes, and makes final product calls. Do not make CEO the default message relay for every working detail.

Adjacent roles should collaborate directly:

- Product Designer <-> Product Developer for feasibility, missing states, UX behavior, and implementation-ready design.
- Product Developer <-> QA Verifier for ready-for-test handoff, failures, reproduction steps, fixes, and rechecks.

Escalate back to CEO only when user intent, scope, priority, final acceptance, release risk, or cross-role conflict needs a decision.

## Core Workflow

1. Identify the product goal, current workspace, and whether the user wants new threads or management of existing threads.
2. Search for existing related Codex threads before creating anything new. Reuse matching threads when their roles are clear.
3. Create or update repo-local shared state under `docs/agents/`.
4. Define the agent roster, role boundaries, write scopes, and escalation rules.
5. Create missing Codex threads only when authorized, using `list_projects` first and `create_thread` with a role bootstrap prompt.
6. Set thread titles, then record each thread id in `docs/agents/manifest.md`.
7. Assign product direction through the orchestrator thread, then allow adjacent roles to collaborate directly: Designer <-> Developer and Developer <-> QA.
8. Read role outputs, update shared state, route product decisions, and run a design/development and development/QA feedback loop.

## Tool Routing

Use Codex App thread tools when available:

- `list_threads` to discover existing project threads.
- `read_thread` to inspect recent status and outputs.
- `list_projects` then `create_thread` to create role threads.
- `set_thread_title` to make role names stable.
- `send_message_to_thread` to assign or relay work.
- `handoff_thread` only when moving another thread between local checkout and worktree is necessary.

If these tools are not already visible, search for thread tools first. Do not emulate thread creation with files or raw instructions.

## Shared State Files

Create `docs/agents/` in the project when it does not exist:

- `manifest.md`: thread ids, names, roles, scopes, status, and current branch/worktree.
- `board.md`: task board with owner, status, dependencies, write scope, and done criteria.
- `handoffs.md`: timestamped cross-agent handoffs and review results.
- `decisions.md`: durable product/architecture decisions.
- `messages.md`: optional log for important cross-agent messages, ACKs, direct collaboration summaries, and protocol updates.

For the exact templates and message formats, read `references/protocol.md`.

## Default Roster

Use the smallest useful team. Start with these roles for product builds:

- `CEO / Product Owner`: current thread; talks with the user, owns product intent, priorities, scope decisions, and final acceptance.
- `Product Designer`: product spec, UX flows, copy, constraints, and acceptance criteria.
- `Product Developer`: implementation, refactoring, and local verification in assigned code scopes.
- `QA Verifier`: tests, acceptance checks, screenshots, regression risks, and bug reports.

Add specialized roles only when needed:

- `Tech Architect`: architecture, module boundaries, desktop/runtime tradeoffs.
- `Researcher`: official docs, market/reference research, dependency behavior.
- `UI Designer`: visual design system and interaction polish.
- `Git Steward`: branch/commit/PR hygiene for larger efforts.

## Thread Creation Policy

Before creating new threads:

1. Run `list_threads` with the product or project name.
2. Classify existing threads by title, cwd, preview, and status.
3. Reuse threads with matching roles unless the user asked for fresh agents.
4. Create only missing roles.

When creating threads:

- Use `list_projects` first and choose the current project when possible.
- Prefer `target.type="project"` with `environment.type="local"` for planning and coordination roles that only read/write shared docs.
- Prefer `environment.type="worktree"` for independent code-changing roles that may run in parallel.
- Avoid multiple agents writing the same files. Put the allowed write scope in both the task prompt and `board.md`.
- After creation, set a clear title such as `<product> / Product Developer`.

## Scheduling Model

Use staged orchestration:

1. `Intake`: capture product goal, constraints, users, and non-goals.
2. `Design`: Product Designer drafts or updates product spec and acceptance criteria.
3. `Architecture`: Tech Architect or Orchestrator defines module boundaries and technical plan.
4. `Implementation`: Product Developer works from assigned tasks and write scopes.
5. `Verification`: QA Verifier checks behavior, tests, screenshots, and regressions.
6. `Fix Loop`: Product Developer and QA Verifier work directly on failures, fixes, and rechecks inside the original scope.
7. `Release Summary`: Orchestrator reports changed files, evidence, decisions, and remaining risks.

Do not make the orchestrator a bottleneck for normal working detail. Product Designer and Product Developer should interact directly on feasibility and implementation-ready design. Product Developer and QA Verifier should interact directly on verification, findings, fixes, and rechecks. The orchestrator joins when user intent, scope, priority, final acceptance, or cross-role conflict needs a decision.

## Task Assignment Rules

Each assignment must include:

- Task id.
- Owner role and thread id.
- Context files to read.
- Exact objective.
- Allowed write scope.
- Out-of-scope items.
- Done criteria.
- Required final response format.

For implementation tasks, tell agents they are not alone in the codebase, must not revert unrelated changes, and must adapt to concurrent edits. Tell adjacent roles to use the direct interaction formats from `references/protocol.md` when they need design/development or development/QA back-and-forth.

## Completion Criteria

Before telling the user the multi-agent workflow is complete:

- `docs/agents/manifest.md` reflects current thread ids and roles.
- `docs/agents/board.md` has no active unowned critical tasks.
- Handoffs and decisions are recorded.
- Code-changing work has verification evidence or an explicit verification gap.
- QA findings are resolved through the direct Developer<->QA loop, deferred with rationale, or returned to the user as a decision.
