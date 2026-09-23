---
name: astra-orchestrator
description: "Explicit-invocation-only Astra-led native Codex orchestration. Use only when the user explicitly invokes $astra-orchestrator or unmistakably asks to enable this named skill. Split work by difficulty and dependencies across GPT-6 Sol, Terra, and GPT-6 Luna, then use a fresh GPT-6 Sol High subagent for independent audit."
---

# Astra Orchestrator

Run this workflow only after an explicit `$astra-orchestrator` invocation. Preserve the user's scope, permissions, prohibitions, and task-specific settings.

## Resolve Runtime Settings

Read [`config/routing.yaml`](config/routing.yaml) as the persistent policy source. Apply overrides in this order, subject to host instructions and permissions:

1. explicit setting for a specific subtask;
2. explicit setting for this session or whole task;
3. applicable project configuration;
4. persistent settings in `routing.yaml`;
5. Skill defaults.

More specific settings win within one source. Temporary wording such as “this time” never edits persistent files. “Restore defaults” clears overrides in the stated scope. Automatic routing selects the worker model. It must not automatically change reasoning effort unless the user explicitly enables that behavior.

Before orchestration, determine the actual parent model/reasoning information exposed by the host. Loading this Skill does not switch the running parent. If Astra is required but the current parent is not confirmed as Astra, state the limitation and give the relevant launch/new-session instruction; do not claim that prompt text changed the model.

## Route Bounded Tasks

Before dispatch, split the requested deliverable into bounded tasks with inputs, dependencies, acceptance criteria, and file ownership. Assess each task's difficulty, ambiguity, risk, and verification needs, then assign its worker separately. Parallelize independent tasks and sequence dependent ones. Keep tightly coupled work together; do not route a mixed task wholesale to one model just because one part is difficult. The parent may retain small coordination and integration steps.

Read [`references/roles-and-routing.md`](references/roles-and-routing.md) when selecting roles or splitting a mixed task. In brief:

- Sol handles difficult reasoning, architecture, non-trivial algorithms, subtle cross-module faults, and high-impact analysis.
- Luna is eligible only when scope and rules are clear, ambiguity and risk are low, reasoning is simple, and verification is direct.
- Terra handles the remaining ordinary engineering, analysis, data, documentation, debugging, and implementation tasks.
- Astra alone owns interpretation, decomposition, permissions, dependencies, integration, conflict resolution, acceptance, and the final answer.
- After integration, a newly created GPT-6 Sol High auditor independently examines the stable result. It is separate from all implementation workers.

Do not delegate merely to use every role. Astra may directly handle short clarification, routing, integration, or work that cannot be separated effectively.

## Use Real Native Delegation

Read [`references/runtime-and-overrides.md`](references/runtime-and-overrides.md) before the first dispatch or when applying overrides.

For every real worker, call the host's native subagent/delegation tool. With the currently supported `spawn_agent` interface:

- set `task_name` to `astra_orch_sol_<task-id>`, `astra_orch_terra_<task-id>`, `astra_orch_luna_<task-id>`, or `astra_orch_audit_<task-id>`;
- set `model` to the resolved exact model ID;
- set `reasoning_effort` to the resolved effort;
- use a context mode that permits those explicit overrides;
- instruct the worker not to spawn subagents.

Never simulate workers in one thread, relabel an inherited worker in prose, use hidden parameters, or silently substitute a model after a failure. If the runtime does not expose model/effort selection, report the concrete limitation rather than asserting a model identity.

Track `requested`, `resolved`, `submitted`, and `actual` separately. When the host does not report actual identity, use `actual_model: UNKNOWN` and/or `actual_reasoning: UNKNOWN`; worker self-description is not evidence.

## Delegate Safely

Use [`templates/task-packet.yaml`](templates/task-packet.yaml) as a lightweight checklist, not mandatory bureaucracy. Read [`references/delegation-and-review.md`](references/delegation-and-review.md) for concurrency, dependencies, ownership, retries, evidence, and integration rules.

Essential invariants:

- only Astra creates or manages workers; worker depth is at most one;
- use no more than the resolved parallel limit and any stricter host limit;
- parallelize only independent work;
- every write task has exclusive `owned_paths`; workers may not expand scope;
- no Best-of-N or duplicate full implementations by default;
- wait for writers to finish before integration;
- allow at most one reasoned reassignment per task by default;
- complexity escalation does not change reasoning effort unless explicitly authorized.

Use [`templates/worker-result.yaml`](templates/worker-result.yaml) to normalize results. Verify claims against artifacts and evidence. Run project-appropriate validation when authorized by the current task; the installation-time no-test restriction that created this Skill is not a permanent ban on future project validation.

## Finish as Astra

Inspect relevant outputs and changes, reconcile conflicts, and run authorized integration checks. Before final delivery of substantive work, create a fresh read-only audit subagent using `audit.model` and `audit.reasoning` from `config/routing.yaml` (defaults: `gpt-6-sol`, `high`) and `fork_turns="none"`. Give it the original requirements, applicable constraints, stable artifacts or diff, and validation evidence. Do not reuse an implementation worker as auditor. The auditor must not modify files or spawn agents.

Follow the independent audit and finding-closure procedure in [`references/delegation-and-review.md`](references/delegation-and-review.md). Astra evaluates the findings, coordinates justified fixes, verifies their effects, and owns final acceptance. Distinguish completed checks from unexecuted checks. If audit is unavailable or incomplete, report that limit without claiming independent approval. Explicit user overrides, including no-subagent instructions, take precedence. Merely discussing or editing this skill does not activate its workflow.
