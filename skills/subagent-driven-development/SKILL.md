---
name: subagent-driven-development
description: Use when executing an implementation plan with mostly independent tasks via one implementer subagent per task plus reviews.
---

# Subagent-Driven Development (lean)

Run a plan by dispatching one fresh implementer subagent per task, a task
review after each (spec compliance + code quality), and one whole-branch
review at the end. Subagents get file paths and a short brief, never your
history; you keep your own context for coordination.

## Hard limits (read these first; they survive compaction)

1. **Fix rounds: 3 per task, maximum.** Then adjudicate (see Breaker). Never a 4th.
2. **Every fix round dispatches a FRESH implementer** with brief path, report
   path and the open findings. Do not resume a finished implementer with
   follow-up messages: resumed workers keep their whole history and grow to
   300k+ tokens (upstream #2551).
3. **Always set `model` on every dispatch.** Omitted = the session's most
   expensive model. Defaults below.
4. **One implementer at a time.** Never parallel implementers on one branch.
5. **Subagents never dispatch subagents** (implementers and reviewers alike).
6. **Short returns.** Implementers and reviewers write full reports to files and
   return at most 15 lines. Nothing long is pasted into your context.
7. **Phases.** Plans over 8 tasks run in phases of at most 6 tasks. At a phase
   boundary, update the ledger and end your turn with one line: "Phase N done;
   `/clear` then 'resume <plan path>' to continue cheaply." Skip this only if
   your human partner said to run straight through.

## Model defaults

| Role | Default model |
|------|---------------|
| Implementer, plan contains the full code (transcription + tests) | `haiku` |
| Implementer, prose task, 1-3 files | `sonnet` |
| Implementer, multi-file integration or design judgment | session's top model |
| Task reviewer and scoped re-reviewer | `sonnet` (`haiku` for tiny mechanical diffs) |
| Fix round 3 implementer | one tier above the previous implementer |
| Final whole-branch review | session's top model |

Turn count matters as much as price: the cheapest tier can take 2-3x the turns
on multi-step prose tasks, so keep `sonnet` as the floor for those.

## Continuous execution and rulings

Execute tasks without check-ins between them (phase boundaries excepted).
Decide conflicts and ambiguities yourself; the spec is the binding authority,
the plan is its argument. Record each decision in the ledger as
`Ruling: <decision> — <why> — <cost if wrong>`. Stop and ask only for: an
irreversible or destructive operation; a security-sensitive action; a side
effect outside this worktree (merge, push to a shared branch, publish); or a
plan so broken every path forward is a guess.

## Setup

- Isolated workspace: superpowers-lean:using-git-worktrees. Never implement on
  main/master without explicit consent.
- Run `bash scripts/sdd-workspace PLAN_FILE` (this skill's directory). It prints
  the plan's git-ignored workspace under `<repo>/.superpowers/sdd/`.
- Ledger at `<workspace>/progress.md`, first line
  `# SDD ledger — plan: <plan path>`. If it exists and names your plan, tasks
  with `Task <N>: complete` are DONE: resume at the first task without one.
  A ledger naming another plan is not yours; leave it.
- After compaction or `/clear`, trust the ledger and `git log` over memory.
  Re-dispatching completed tasks is the most expensive failure there is.
- Read the plan once (and its Spec, if named). One todo per task.
- Pre-flight: scan once for tasks that contradict each other or the Global
  Constraints, and for plan mandates the review rubric would call defects.
  Write findings and your rulings to the ledger, then start Task 1.

## The task loop

**Batch tiny same-shape tasks** (the same small edit across files) into one
dispatch and one review.

**Hand artifacts over as files.** Everything pasted into a dispatch and
everything a subagent prints back stays in your context and is re-read on
every later turn.

**Waiting:** don't poll with short timeouts. Do local bookkeeping while
children run; when idle, wait in bounded stretches and post one status line
between them.

### 1. Dispatch the implementer

- Record BASE (`git rev-parse HEAD`).
- `bash scripts/task-brief PLAN_FILE N` writes the task text to a file and
  prints its path. The dispatch contains only: one line on where the task
  fits; the brief path ("read this first; it is your requirements"); interfaces
  or decisions from earlier tasks the brief can't know; your resolution of any
  ambiguity; the report path (`…/task-N-report.md`) and report contract; any
  ledger pointer to a parked finding in the same area.
- Never paste prior-task summaries or the plan. Never make a subagent read the
  whole plan file.
- Template: [implementer-prompt.md](implementer-prompt.md).

### 2. Handle the report

- **DONE:** `bash scripts/review-package PLAN_FILE BASE HEAD` (BASE as recorded,
  never `HEAD~1`) and dispatch the reviewer with the printed path.
- **DONE_WITH_CONCERNS:** read the concerns; fix correctness/scope doubts
  before review, note observations and proceed.
- **NEEDS_CONTEXT:** supply it and re-dispatch fresh.
- **BLOCKED:** more context → same model; more reasoning → higher tier; too
  big → split; plan wrong → rule, ledger, re-dispatch with the ruling.
  Never retry unchanged.

### 3. Review the task

- Reviewer inputs: brief path, report path, review-package path, and the
  binding Global Constraints copied verbatim. Nothing else.
- Don't ask the reviewer to re-run tests the implementer ran, don't add
  open-ended directives, and never pre-judge findings ("do not flag…").
- The reviewer writes its full review to `…/task-N-review.md` and returns the
  short verdict block only.
- Resolve every "⚠️ cannot verify from diff" item yourself; a confirmed gap
  is a failed spec review.
- **Light path:** a task whose plan text contains the complete code, touching
  at most 2 files, with tests passing, may be reviewed together with up to 2
  following light tasks in one review package (BASE of the first, HEAD of the
  last). Everything else gets its own task review.
- Template: [task-reviewer-prompt.md](task-reviewer-prompt.md).

### 4. The fix loop (max 3 rounds)

Triggers: spec ❌, any Critical/Important finding, or a confirmed ⚠️ gap.

- Minor findings go to the ledger (`Task <N>: minor (deferred): …`) and to the
  final reviewer; they never enter the loop.
- Findings that conflict with plan text are yours to rule on, with a ledger
  ruling, before any fix.
- **Each round** = one FRESH implementer (brief path, report path, findings
  verbatim; rounds 1-2 same tier, round 3 one tier up, framed "a prior
  implementer tried N times; read the report file") + one scoped re-review:
  `bash scripts/review-package PLAN_FILE FIX_BASE HEAD` with
  [re-review-prompt.md](re-review-prompt.md). The fixer appends its fix report
  (covering tests, command, output) to the same report file.
- Ledger after each round:
  `Task <N>: fix round <R>/3 (<X> addressed, <Y> open — …; commits a..b)`.
- Never fix findings yourself in the controller.

**Breaker (after round 3):** stop dispatching and adjudicate each open
finding. Wrong or contestable → park with a ruling. Real but nothing builds
on it → park as deferred. Real and load-bearing → rule on the smallest
unblocking change and carry it into the next dispatch. Every adjudication is a
ledger line; silent discards are forbidden.

### 5. Complete the task

Ledger `Task <N>: complete (commits base..head, review clean | K parked)`,
mark the todo done, move on. Never advance with open Critical/Important
findings that are neither fixed nor parked with a ruling.

## Final review

`bash scripts/review-package PLAN_FILE MERGE_BASE HEAD`, then dispatch ONE
reviewer on the session's top model using
[code-reviewer.md](../requesting-code-review/code-reviewer.md), pointing it at
the ledger's deferred and parked lines. If it returns findings: ONE fix
dispatch with the full list (not one per finding), one scoped re-review,
adjudicate residuals. No second fix wave.

## Finish

Collect every ledger line containing `Ruling:` into your final message under
"Rulings I made", in order, each with its cost if wrong. Then delete this
plan's workspace (`rm -rf <workspace>`; never sibling plans' directories) and
use superpowers-lean:finishing-a-development-branch.

## Red flags

| Thought | Reality |
|---------|---------|
| "I'll resume the implementer, it already has context" | Its context is the cost. Dispatch fresh with the report file. |
| "One more round will converge" | Three is the cap. Adjudicate. |
| "I'll fix it myself" | Controller fixes skip review and bloat your context. |
| "No need to set a model" | Omitted model = most expensive model. |
| "I'll paste the previous tasks' summary for context" | Pass interfaces only. The rest is history the subagent re-reads every turn. |
| "Skip the re-review, the fix was small" | Small re-reviews are cheap (`sonnet`/`haiku`); unreviewed fixes are how regressions land. |
