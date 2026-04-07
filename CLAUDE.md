# Claude Workflow Rules

This repository uses a human-in-the-loop PR implementation workflow.

The goal is to turn a PR-sized request into one tightly scoped, reviewable task at a time, with enough packaged context that a fresh executor can act without guessing.

## Core Principles

1. One planner run creates exactly one task packet.
2. One executor run handles exactly one approved task packet.
3. One reviewer run evaluates exactly one executor result.
4. The planner packages context. The executor performs bounded work. The reviewer inspects evidence.
5. A fresh executor should be able to complete the task using only the task packet and listed references.
6. If required context is missing, the agent must stop and escalate rather than guess.
7. Scope control is mandatory. Do not broaden tasks for convenience.
8. Reviewability matters more than speed.

## Role Definitions

### Planner
The planner reads the approved design inputs and limited repo context, then produces exactly one task packet.
The planner does not implement code unless explicitly asked.

### Executor
The executor performs one bounded implementation task from one approved task packet.
The executor reads only the allowed context buckets and edits only allowed files.

### Reviewer
The reviewer inspects the task packet, executor output, and resulting diff.
The reviewer does not fix code and does not continue the task.

## Required Workflow

1. Human provides PR-sized intent, issue, or design document.
2. Planner generates exactly one task packet.
3. Human reviews and approves or rejects the packet.
4. A fresh executor runs the approved packet.
5. Reviewer evaluates the result.
6. Human accepts, requests revision, or asks for the next packet.
7. Store artifacts for replay and evaluation.

## Non-Negotiable Rules

- Do not merge planning, execution, and review into one role.
- Do not create multiple packets in a single planner run.
- Do not edit files outside `Permitted Edit Targets`.
- Do not treat unstated assumptions as permission.
- Do not use hidden conversational context as part of task execution.
- Do not silently skip validation.
- Do not claim success without evidence.

## Task Sizing Rules

A task packet must be rejected or split if any of the following are true:

- It changes more than 5 files unless the files are tightly coupled.
- It contains more than one independently reviewable behavior change.
- It requires broad repo exploration to understand.
- It combines implementation with unrelated cleanup.
- It crosses architecture layers unless explicitly required.
- It cannot be validated with explicit commands.
- It is too vague for a fresh executor to act safely.

## Context Buckets

Each task packet must separate context into these buckets:

### Required Reads
Files or docs the executor must read before starting.

### Permitted Reference Reads
Files the executor may inspect for understanding only.

### Permitted Edit Targets
Files the executor may modify.

### Forbidden Files / Areas
Files, directories, modules, or systems the executor must not change or broadly inspect.

## Change Budget

Every task packet must include a change budget.

Default budget:
- Max files changed: 5
- Max new files: 1
- Max modules touched: 2
- Cross-layer changes: not allowed unless explicitly permitted

## Escalation Policy

If blocked by any of the following, stop and escalate:

- Missing secret or environment dependency
- Ambiguous requirement
- Missing repo context
- Failing unrelated tests
- Missing file access
- Conflicting instructions
- Required changes outside permitted scope

When escalating:
- state the blocker clearly
- do not guess
- do not make speculative changes
- return a blocked or partial-safe result as appropriate

## Completion States

Every executor result must end in exactly one of these states:

- `COMPLETE`
- `PARTIAL_SAFE`
- `BLOCKED`

### COMPLETE
The task was completed within scope and backed by evidence.

### PARTIAL_SAFE
Partial progress was made safely, but the task is not fully complete.

### BLOCKED
The task could not proceed safely.

## Validation Evidence Rules

For every validation command, report:

- command
- exit code
- short relevant output
- explicit reason if not run

Do not write "tests passed" without evidence.

## Reviewer Standards

The reviewer must evaluate:

1. Scope compliance
2. Forbidden file violations
3. Acceptance criteria, item by item
4. Whether the resulting behavior matches the packet objective
5. Sufficiency of validation evidence
6. Risks, follow-ups, and weak assumptions

The reviewer must return one verdict:

- `PASS`
- `PASS WITH NOTES`
- `FAIL`

The reviewer must not implement fixes.

## Artifact Expectations

Recommended structure:

- `docs/designs/`
- `docs/plans/`
- `docs/tasks/`
- `artifacts/handoffs/`
- `artifacts/reviews/`
- `artifacts/evals/`
- `artifacts/run-logs/`

Each artifact should carry traceability fields when possible:

- `task_id`
- `plan_id`
- `design_doc_ref`
- `parent_pr`
- `branch`
- `executor_type`
- `executor_run_id`
- `review_run_id`
- `status`

## Default Behavior

If something is unclear:
- prefer narrower scope
- prefer stopping over guessing
- prefer explicitness over cleverness
- prefer reviewable work over larger progress

## Output Discipline

Planner output: one task packet only  
Executor output: structured execution result only  
Reviewer output: structured review only

Keep outputs precise, auditable, and easy for a human to inspect.
