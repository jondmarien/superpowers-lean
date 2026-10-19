---
name: using-superpowers
description: Bootstrap rules for superpowers-lean skills. Already injected at session start; invoke manually only if it is missing.
disable-model-invocation: true
---

<SUBAGENT-STOP>
If you were dispatched as a subagent for a specific task, ignore this and do the task.
</SUBAGENT-STOP>

**Superpowers-lean skills** (load with the Skill tool, prefix `superpowers-lean:`):
brainstorming (new features/behavior, before code) · writing-plans (spec → plan) ·
subagent-driven-development / executing-plans (run a plan) · test-driven-development
(any feature or bugfix) · systematic-debugging (any bug or test failure, before fixing) ·
verification-before-completion (before claiming done) · requesting-code-review /
receiving-code-review · using-git-worktrees · finishing-a-development-branch ·
dispatching-parallel-agents.

**Rules**
1. When a skill clearly matches the task, invoke it before acting and follow it.
   "Build X" → brainstorming first. "Fix this bug" → systematic-debugging first.
   Process skills come before implementation skills.
2. Skip skills for questions, explanations, lookups, and trivial one-step edits.
3. Load a skill once per context. Do not re-invoke a skill whose content is already
   in this conversation; re-invoke only after compaction if you need its details.
4. Subagents cost a full context each. Dispatch only when a skill calls for it or the
   work is large and isolated; always set `model` explicitly (Haiku for search and
   mechanical edits, Sonnet for implementation and routine review, the session's top
   model only for architecture and the final branch review); hand over file paths,
   not pasted history; ask for short replies with details written to files.
5. User instructions (CLAUDE.md, AGENTS.md, direct requests) override skills.

Platform notes, if needed: `references/<harness>-tools.md` in this skill's directory.
