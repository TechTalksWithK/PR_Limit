# Runbook

This runbook explains how to operate the human-in-the-loop PR implementation workflow in VS Code using Claude.

The workflow is:

1. Human writes or approves a design doc
2. Planner creates one task packet
3. Human reviews and approves the packet
4. Fresh executor runs the packet
5. Reviewer evaluates the result
6. Human decides what happens next

The goal is to turn PR-sized work into one tightly scoped, reviewable task at a time, with context packaged so the executor does not have to guess.

---

## 1. Repository Setup

Create this structure:

- `CLAUDE.md`
- `agents/planner.md`
- `agents/executor-implementer.md`
- `agents/reviewer.md`
- `docs/designs/`
- `docs/plans/`
- `docs/tasks/`
- `artifacts/handoffs/`
- `artifacts/reviews/`
- `artifacts/evals/`
- `artifacts/run-logs/`

Recommended starting files:
- `docs/designs/design-template.md`
- `docs/tasks/TASK-template.md`

---

## 2. Role Definitions

### Human
The human is the gatekeeper.

Responsibilities:
- define or approve the design
- decide whether a task packet is good enough
- decide whether reviewed work is accepted
- decide whether to create the next packet

### Planner
The planner produces exactly one task packet.

The planner should:
- identify the smallest useful next step
- package enough context for a fresh executor
- define scope boundaries clearly
- define measurable acceptance criteria
- define validation and escalation rules

The planner should not:
- implement the code by default
- generate multiple packets in one run
- emit vague tasks that require guessing

### Executor
The executor performs one approved task packet.

The executor should:
- read only what the packet allows
- edit only what the packet allows
- stay within the change budget
- return evidence-backed results

The executor should not:
- broaden scope
- explore unrelated repo areas
- invent requirements
- silently skip validation

### Reviewer
The reviewer evaluates the result conservatively.

The reviewer should:
- check scope compliance
- inspect forbidden-file violations
- verify acceptance criteria one by one
- verify behavior changed as intended
- inspect validation evidence

The reviewer should not:
- fix code
- continue the task
- broaden scope

---

## 3. Daily Operating Flow

### Step 1: Create or update a design doc
Write a design doc in `docs/designs/`.

The design doc should explain:
- the problem
- the intended outcome
- scope and non-goals
- relevant repo areas
- constraints
- risks
- validation ideas

Do not try to turn the design doc into a task packet manually unless needed.
The planner’s job is to convert design into one reviewable task.

### Step 2: Ask the planner for one task packet
Provide the planner:
- the design doc
- any approved issue/PR context
- minimal repo context

The planner must produce exactly one task packet.

Save that output to:
- `docs/tasks/TASK-YYYYMMDD-###.md`

### Step 3: Review the task packet manually
Before execution, confirm:
- the objective is singular
- file scope is explicit
- forbidden areas are explicit
- acceptance criteria are measurable
- validation commands exist
- escalation rules exist
- a fresh executor could act without prior context

If the packet is too broad, reject it and ask the planner to split or tighten it.

### Step 4: Approve the task packet
Once reviewed, mark:
- `Reviewed by human: yes`
- `Approved for execution: yes`

Do not run the executor on an unapproved packet.

### Step 5: Start a fresh executor run
Use a fresh Claude execution context.

Important:
The executor should not rely on earlier chat history.
Give it:
- the approved task packet
- access only to the repo/files it needs
- no hidden assumptions

The executor should return:
- files changed
- summary of changes
- acceptance criteria status
- validation evidence
- risks or follow-ups
- blockers
- completion state

Save the execution result to:
- `artifacts/handoffs/HANDOFF-TASK-YYYYMMDD-###.md`

### Step 6: Run the reviewer
Provide the reviewer:
- the approved task packet
- the executor result
- the diff or changed files
- validation output if available

The reviewer must return:
- verdict
- scope compliance
- forbidden-file check
- acceptance criteria review
- behavior match assessment
- validation evidence summary
- risks or follow-ups
- recommended next action

Save the review result to:
- `artifacts/reviews/REVIEW-TASK-YYYYMMDD-###.md`

### Step 7: Human decision
After review, choose one:

#### Accept
Accept if:
- reviewer verdict is PASS or PASS WITH NOTES
- scope remained controlled
- behavior matches the task objective
- evidence is sufficient

#### Revise and rerun
Use when:
- the task is still the correct scope
- the implementation needs correction
- the packet itself is still valid

#### Split into a smaller packet
Use when:
- the task was too broad in practice
- the executor struggled due to hidden complexity
- the reviewer finds mixed or unrelated changes

#### Escalate blocker
Use when:
- the task cannot proceed safely
- the packet lacked critical context
- required changes are outside permitted scope

---

## 4. How To Judge Packet Quality

A good packet:
- has one specific objective
- names the files to read
- names the files to edit
- names the forbidden areas
- defines measurable acceptance criteria
- includes validation commands
- includes stop conditions
- is small enough to review comfortably

A bad packet:
- says “update the feature” without file boundaries
- mixes implementation and unrelated cleanup
- depends on hidden conversational context
- has vague acceptance criteria like “works properly”
- gives no escalation path
- forces the executor to explore broadly

Rule:
If a fresh executor would need to guess, the packet is weak.

---

## 5. Completion States

### COMPLETE
Use when:
- the requested behavior is implemented
- scope was respected
- validation evidence supports completion

### PARTIAL_SAFE
Use when:
- safe, in-scope progress was made
- the task is not fully done
- the partial work is still reviewable and non-speculative

### BLOCKED
Use when:
- the task cannot proceed safely
- the next move would require guessing
- required changes fall outside the packet contract

---

## 6. Review Standards

A reviewer should inspect:

### Scope compliance
Did the executor stay inside permitted edit targets and change budget?

### Forbidden files
Were any forbidden or unrelated files touched?

### Acceptance criteria
Was each criterion met, not met, or partially met?

### Behavior match
Did the actual change accomplish the packet objective?

### Evidence
Were commands run?
Were exit codes shown?
Was the output relevant?
Were skipped commands explained?

### Risks
Are there hidden assumptions, missing tests, or deferred work?

---

## 7. Common Failure Modes

### Failure: packet is technically singular but too broad
Fix:
- reduce file count
- reduce behavior count
- reduce architecture spread

### Failure: executor guesses
Fix:
- strengthen required reads
- make edit targets explicit
- improve acceptance criteria
- clarify escalation rules

### Failure: reviewer cannot prove success
Fix:
- require better validation evidence
- improve validation commands
- make acceptance criteria more measurable

### Failure: task includes “bonus cleanup”
Fix:
- move cleanup into a separate future packet

### Failure: packet depends on prior conversation
Fix:
- add missing context into the packet itself

---

## 8. Naming Conventions

Suggested file naming:

### Design docs
- `docs/designs/DESIGN-YYYYMMDD-001-short-name.md`

### Task packets
- `docs/tasks/TASK-YYYYMMDD-001-short-name.md`

### Execution results
- `artifacts/handoffs/HANDOFF-TASK-YYYYMMDD-001.md`

### Review results
- `artifacts/reviews/REVIEW-TASK-YYYYMMDD-001.md`

### Eval results
- `artifacts/evals/EVAL-TASK-YYYYMMDD-001.json`

Try to keep IDs stable across the workflow.

---

## 9. Recommended First Pilot

For your first real test, choose a very small task.

Good first pilot examples:
- add one narrow validation check
- update one function’s logging behavior
- add one focused unit test
- adjust one small CI script path
- update one documentation page tied to one code change

Avoid as your first pilot:
- large refactors
- multi-module behavior changes
- dependency upgrades
- cross-layer architectural changes

The purpose of the first pilot is to validate the workflow, not maximize output.

---

## 10. Manual Operator Checklist

Before planner run:
- [ ] design is clear enough to plan
- [ ] non-goals are listed
- [ ] relevant repo areas are identified

Before executor run:
- [ ] packet is singular
- [ ] reads/edits/forbidden areas are explicit
- [ ] acceptance criteria are measurable
- [ ] validation commands exist
- [ ] packet is human-approved

Before reviewer run:
- [ ] executor result is saved
- [ ] diff is available
- [ ] validation output is available if possible

Before marking done:
- [ ] reviewer verdict is acceptable
- [ ] behavior matches objective
- [ ] evidence is sufficient
- [ ] follow-ups are either accepted or deferred explicitly

---

## 11. Practical Operating Advice

- Start with one executor type only: implementer
- Keep packets smaller than you think you need
- Treat packet quality as the main lever
- Prefer stopping over guessing
- Use review failures to improve the template, not just the implementation
- Keep artifacts organized from day one
- Test with fresh execution contexts regularly

The system is working when:
- packets are easy to review
- executors rarely guess
- reviewers can judge results quickly
- humans can decide the next step with low ambiguity