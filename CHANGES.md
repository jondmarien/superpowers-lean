# CHANGES (superpowers-lean vs obra/superpowers v7.0.0 @ bb92a77)

Token counts use tiktoken o200k_base as a proxy (Claude's tokenizer differs by
roughly ±10%; chars/4 gives similar numbers).

## Always-on cost (paid on every request of every main session)

| Item | Upstream | Lean | Saved |
|------|---------:|-----:|------:|
| SessionStart injection (`additionalContext`) | 791 tok (3,405 chars) | 406 tok (1,742 chars) | −385 |
| Skill listing (names + descriptions Claude Code always loads) | 561 tok (15 skills) | 385 tok (12 listed) | −176 |
| **Total always-on per main session** | **1,352** | **791** | **−561 (−41%)** |
| Per subagent (no SessionStart in subagents; listing only) | 561 (+~735 + a turn when it auto-invokes `using-superpowers`) | 385 (cannot auto-invoke it) | −176 to −910 |
| Per compaction cycle: bootstrap re-injection | 791 | 406 | −385 |
| Per compaction cycle: SDD skill re-attached (cap 5k tok/skill) | ~5,000 (truncated; loses the round cap and breaker) | 2,213 (whole file, limits at top) | −2,800, and nothing lost |

## Changes

1. **Compact bootstrap** (`skills/using-superpowers/SKILL.md`, 735 → 435 tok).
   Removed the "1% chance → you MUST invoke" rule, "invoke before ANY response
   including clarifying questions", and the Red Flags table (including "I
   remember this skill → Skills evolve. Read current version", which drives
   re-invocations; each re-invocation appends another full copy of the skill).
   New rules: invoke when a skill clearly matches; skip for questions, lookups
   and trivial edits; load each skill once per context; explicit cheap models
   for subagents; file handoffs.
2. **`using-superpowers` is manual-only** (`disable-model-invocation: true`).
   Its description said "Use when starting any conversation", so the model
   (and subagents, which start conversations too) invoked it even though the
   hook already injected it. Now it is not in the listing and can't be
   auto-invoked; `/using-superpowers` still works by hand.
3. **Hook** (`hooks/session-start`): injects only the skill body (frontmatter
   stripped) in a short wrapper instead of the `<EXTREMELY_IMPORTANT>` preamble;
   exits silently when the input carries `agent_id` (subagent/teammate) or
   `source` is `resume`/`fork`. Matcher stays `startup|clear|compact`:
   compaction drops the original injection, so one small re-injection per
   compaction is needed and does not accumulate.
4. **Shorter descriptions** for 8 skills; `writing-skills` and
   `diagnosing-superpowers` made manual-only (out of the listing; still
   available via `/writing-skills`, `/diagnosing-superpowers`).
5. **subagent-driven-development rewritten** (7,533 → 2,213 tok, 32k → 9k chars,
   so the whole file survives the ~20k-char compaction re-attach; upstream
   #2481). Hard limits first:
   - fix rounds capped at 3 (was 5), then adjudicate;
   - every fix round is a FRESH implementer with brief/report/findings, never a
     resumed one (upstream #2551: resumed implementers grew to 300k+ context and
     were ~48% of subagent tokens);
   - explicit model table: Haiku for full-code transcription tasks and tiny
     re-reviews, Sonnet for prose implementation and task reviews, top model
     only for design-heavy tasks and the final branch review;
   - reviewers and implementers write details to files and return ≤15 lines
     (upstream #1930);
   - plans over 8 tasks run in phases of ≤6 with a `/clear` + "resume" point
     (controller context in #2551 grew 62k → 603k with no compaction);
   - "light path": up to 3 consecutive full-code ≤2-file tasks share one review.
6. **Templates**: task reviewer writes the full review to `[REVIEW_FILE]` and
   returns a short verdict block (no strengths/passed checks in the reply);
   re-reviewer ≤20 lines; implementer template gains fix-round and
   "keep your context small" sections (pipe long output to files, view only
   the screenshots you need), and no longer says it will be resumed.
7. **dispatching-parallel-agents**: at most 3 agents in flight, explicit models,
   short briefs, ≤10-line returns, no nested dispatch, do it yourself below ~3
   domains.
8. **requesting-code-review**: explicit model (Sonnet for task diffs), diff
   handed over as a file, short return.
9. **brainstorming**: builder check runs on Sonnet with the doc path only, and
   is skipped for small changes approved in chat.
10. **writing-plans**: medium-sized tasks; phase headings for plans over 8
    tasks; recommend inline execution for ≤3-task plans.
11. **executing-plans**: dropped the graphviz diagram and the example
    transcript (4,904 → 3,548 tok, under the compaction re-attach cut).
12. **claude-code-tools.md**: documents `CLAUDE_CODE_SUBAGENT_MODEL` and the
    `Explore` agent for cheap search.
13. **Packaging**: renamed to `superpowers-lean` (skills are
    `superpowers-lean:*`), local `marketplace.json` (`superpowers-lean-local`),
    Cursor and OpenCode manifests renamed; Codex/Gemini/Pi/Hermes/Antigravity/
    Devin/Kimi/Muse manifests, tests, docs and release notes dropped.
    LICENSE kept unchanged; NOTICE added.

## Unchanged

TDD, systematic-debugging, verification-before-completion,
receiving-code-review, using-git-worktrees, finishing-a-development-branch:
bodies untouched apart from the namespace rename (and shorter descriptions
where listed). SDD helper scripts unchanged.
