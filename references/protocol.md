# Product Agent Orchestrator Protocol

Use these templates when coordinating product-building Codex threads.

## Collaboration Model

Use a product-chain model:

```text
User <-> CEO / Product Owner <-> Product Designer <-> Product Developer <-> QA Verifier
```

Default to this model for product development unless the user explicitly requests a different coordination style.

The CEO is product-facing, not a universal message switchboard. CEO owns user conversation, product intent, priority, scope decisions, and final acceptance. Adjacent roles collaborate directly:

- Product Designer <-> Product Developer for feasibility, missing states, UX behavior, and implementation tradeoffs.
- Product Developer <-> QA Verifier for ready-for-test handoff, reproduction steps, bug fixes, and rechecks.

Escalate to CEO when user intent, scope, priority, final acceptance, or cross-role conflict needs a decision.

## Shared State Templates

### `docs/agents/manifest.md`

```md
# Agent Manifest

Product: <product name>
Workspace: <absolute project path>
Orchestrator Thread: <thread id>
Last Updated: <YYYY-MM-DD HH:mm TZ>

| Role | Thread Title | Thread ID | Status | Scope | Environment | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| CEO / Product Owner | <title> | <id> | active | user conversation, product intent, priorities, final acceptance | local | Talks with the user and decides scope or priority conflicts. |
| Product Designer | <title> | <id> | idle | product spec, UX, copy, acceptance criteria | local | Collaborates directly with Product Developer. |
| Product Developer | <title> | <id> | idle | implementation, tests, technical verification | local/worktree | Collaborates directly with Product Designer and QA Verifier. |
| QA Verifier | <title> | <id> | idle | acceptance checks, regression review, QA evidence | local/worktree | Collaborates directly with Product Developer. |
```

### `docs/agents/board.md`

```md
# Agent Board

| ID | Task | Owner | Status | Depends On | Write Scope | Done Criteria |
| --- | --- | --- | --- | --- | --- | --- |
| T-001 | Clarify product spec | Product Designer | doing | none | docs/product-spec.md | Design is ready for developer review |
| T-002 | Implement task core | Product Developer | blocked | T-001 | src/features/tasks, tests/unit | Unit tests pass and QA receives handoff |
| T-003 | Verify MVP flow | QA Verifier | blocked | T-002 | read-only | QA RESULT returned with evidence |
```

Status values: `todo`, `doing`, `ready-for-dev`, `implementing`, `ready-for-qa`, `verifying`, `blocked`, `done`, `deferred`.

### `docs/agents/messages.md`

```md
# Inter-Agent Messages

## <YYYY-MM-DD HH:mm TZ> - <Short Subject>

From: <role/thread id>
To: <role/thread id>
Subject: <short topic>

Message:
<important message, ACK, protocol update, or direct collaboration summary>

Outcome:
<pending|acknowledged|resolved|blocked>
```

### `docs/agents/handoffs.md`

```md
# Agent Handoffs

## <YYYY-MM-DD HH:mm TZ> - <From Role> -> <To Role>

Task: <T-###>
Status: <done|blocked|needs-review|ready-for-qa>
Summary:
- <concise result>

Evidence:
- <tests, screenshots, files, commands, or thread summary>

Changed Files:
- <path>

Requests:
- <what the next role must do>

Risks:
- <known gap or none>
```

### `docs/agents/decisions.md`

```md
# Decisions

## D-001 - <Decision Title>

Date: <YYYY-MM-DD>
Owner: <role>
Status: <proposed|accepted|rejected|superseded>

Decision:
<one paragraph>

Rationale:
- <constraint or reason>

Rejected:
- <alternative> | <why rejected>

Review Trigger:
- <when to revisit>
```

## Role Bootstrap Prompts

Use these as initial prompts when creating or repurposing a thread. Replace bracketed values.

### CEO / Product Owner

```md
You are the CEO / Product Owner agent for [PRODUCT].

Responsibilities:
- Discuss ideas with the user.
- Clarify product intent, scope, priorities, and final acceptance.
- Assign product design work to Product Designer.
- Join role discussions only when user intent, scope, priority, final acceptance, or cross-role conflict needs a decision.
- Keep docs/agents/manifest.md, docs/agents/board.md, and docs/agents/decisions.md current.

Do not become a bottleneck for normal design/development or development/QA back-and-forth.
```

### Product Designer

```md
You are the Product Designer agent for [PRODUCT].

Read first:
- docs/product-spec.md if it exists
- docs/development-guide.md if it exists
- docs/agents/manifest.md
- docs/agents/interaction-protocol.md
- docs/agents/board.md

Responsibilities:
- Define product goals, users, non-goals, UX flows, copy direction, and acceptance criteria.
- Protect product taste and scope.
- Collaborate directly with Product Developer on feasibility, missing states, and implementation-ready behavior.
- Escalate to CEO when user intent, scope, or priority is ambiguous.

Final response format:
DESIGN UPDATE
Role: Product Designer
Task:
Status:
Summary:
Files changed:
Developer interaction:
Decisions:
Requests for CEO:
Risks:
```

### Product Developer

```md
You are the Product Developer agent for [PRODUCT].

Read first:
- docs/product-spec.md
- docs/development-guide.md
- docs/agents/manifest.md
- docs/agents/interaction-protocol.md
- docs/agents/board.md
- docs/agents/handoffs.md

Responsibilities:
- Implement assigned tasks inside the allowed write scope.
- Collaborate directly with Product Designer on feasibility and unclear behavior.
- Collaborate directly with QA Verifier on verification, bugs, fixes, and rechecks.
- Add or update focused tests when behavior changes.

Collaboration rules:
- You are not alone in this codebase.
- Do not revert unrelated changes.
- Do not edit outside your assigned write scope unless you report the need first.

Final response format:
DEV UPDATE
Role: Product Developer
Task:
Status:
Summary:
Files changed:
Designer interaction:
QA interaction:
Verification:
Requests for CEO:
Risks:
```

### QA Verifier

```md
You are the QA Verifier agent for [PRODUCT].

Read first:
- docs/product-spec.md
- docs/development-guide.md
- docs/agents/manifest.md
- docs/agents/interaction-protocol.md
- docs/agents/board.md
- docs/agents/handoffs.md

Responsibilities:
- Verify completed work against acceptance criteria.
- Collaborate directly with Product Developer on reproduction steps, failed cases, fixes, and rechecks.
- Escalate to CEO when a failure is caused by product ambiguity or scope.

Final response format:
QA RESULT
Role: QA Verifier
Task:
Verdict: pass|fail|blocked|partial
Evidence:
Findings:
Developer interaction:
Regression risk:
CEO decision needed:
```

## Direct Interaction Formats

### Design To Developer

```md
DESIGN -> DEVELOPMENT
Product: <name>
Task ID: <T-###>
From: Product Designer / <thread id>
To: Product Developer / <thread id>
Subject: <short topic>

Design intent:
<what experience should feel like>

Required behavior:
- <observable behavior>

Acceptance criteria:
- <what must be true>

Open implementation questions:
- <questions for developer, or "none">

Please respond with feasibility, implementation plan, risks, and product questions.
```

### Developer To Designer

```md
DEVELOPMENT -> DESIGN
Product: <name>
Task ID: <T-###>
From: Product Developer / <thread id>
To: Product Designer / <thread id>
Subject: <short topic>

Feasibility:
<yes|partial|blocked>

Implementation plan:
- <planned code/modules>

Tradeoffs:
- <what affects the design>

Questions:
- <questions needing design answer, or "none">

Decision needed from CEO:
- <yes/no and why>
```

### Developer To QA

```md
DEVELOPMENT -> QA
Product: <name>
Task ID: <T-###>
From: Product Developer / <thread id>
To: QA Verifier / <thread id>
Subject: Ready for verification

Implemented:
- <what changed>

Changed files:
- <path>

Verification already run:
- <command or check>

Acceptance criteria to verify:
- <criterion>

Known risks:
- <risk or "none">

Please verify and return `QA RESULT`.
```

### QA To Developer

```md
QA RESULT
Product: <name>
Task ID: <T-###>
From: QA Verifier / <thread id>
To: Product Developer / <thread id>
Verdict: pass|fail|partial|blocked

Evidence:
- <commands, screenshots, files, or observations>

Findings:
- <finding, severity, reproduction steps>

Fix request:
- <specific fix needed, or "none">

CEO decision needed:
- <yes/no and why>
```

## Completion Criteria

Before reporting a multi-agent product task complete:

- CEO/user intent is captured.
- Design-to-development handoff exists when design shaped implementation.
- Development-to-QA handoff exists for code-changing work.
- QA result exists with evidence.
- Product or scope decisions are recorded.
- Open blockers are resolved, deferred with rationale, or returned to the user.
