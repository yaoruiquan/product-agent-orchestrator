# Product Agent Orchestrator

A Codex skill for coordinating product work across role-specific agent threads.

It uses a simple product-chain model:

```text
User <-> CEO / Product Owner <-> Product Designer <-> Product Developer <-> QA Verifier
```

The CEO / Product Owner owns user intent, priorities, scope, and final acceptance. Designer, Developer, and QA collaborate directly on normal working details so the orchestrator does not become a bottleneck.

## What It Does

- Creates or reuses role-specific Codex threads for product work.
- Defines clear role boundaries, write scopes, task ownership, and escalation rules.
- Maintains shared project state under `docs/agents/`.
- Provides handoff formats for design, development, QA, decisions, and cross-agent messages.
- Keeps multi-agent workflows grounded in task boards, evidence, and verification.

## When To Use

Use this skill when you want to turn a product idea into a managed Codex product team, especially when work needs separate product, design, implementation, and QA responsibilities.

Do not use it for simple advice or one-off implementation tasks. In those cases, a normal plan or direct coding workflow is usually enough.

## Install

Place this directory at:

```text
~/.codex/skills/product-agent-orchestrator
```

Expected files:

```text
SKILL.md
references/protocol.md
agents/openai.yaml
```

## Usage

Invoke the skill in Codex:

```text
$product-agent-orchestrator
```

The skill will inspect the current product/workspace, look for existing related threads, create or update `docs/agents/`, and coordinate role-specific work.

## Shared State

The skill uses these project-local files:

- `docs/agents/manifest.md`
- `docs/agents/board.md`
- `docs/agents/handoffs.md`
- `docs/agents/decisions.md`
- `docs/agents/messages.md`

See `references/protocol.md` for templates and message formats.

