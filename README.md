# superpowers-lean

Token-lean fork of [obra/superpowers](https://github.com/obra/superpowers) v7.0.0
(commit `bb92a77`). Same workflows (brainstorming, writing/executing plans,
subagent-driven development, TDD, systematic debugging, code review, git
worktrees, verification, finishing a branch) with a smaller bootstrap and
cheaper, bounded subagent use. See `CHANGES.md` for every change and its
token effect, and `NOTICE`/`LICENSE` for attribution (MIT).

Skills are namespaced `superpowers-lean:<skill>`. **Uninstall or disable the
upstream `superpowers` plugin first**, or both bootstraps and both skill sets
load.

## Install: Claude Code (Windows, macOS, Linux)

1. Put this folder somewhere permanent, e.g. `C:\Users\<you>\plugins\superpowers-lean`.
2. In Claude Code:
   ```
   /plugin uninstall superpowers@<marketplace>     (run /plugin to see the exact id)
   /plugin marketplace add C:\Users\<you>\plugins\superpowers-lean
   /plugin install superpowers-lean@superpowers-lean-local
   ```
   Restart Claude Code. `/plugin` should list `superpowers-lean` as enabled.
3. Windows needs Git for Windows (bash) for the SessionStart hook, same as upstream.
4. To update after editing the folder: `/plugin marketplace update superpowers-lean-local`.

### Recommended Claude Code settings (biggest savings, outside the plugin)

Add to `~/.claude/settings.json` (`%USERPROFILE%\.claude\settings.json`):

```json
{
  "env": {
    "CLAUDE_CODE_SUBAGENT_MODEL": "sonnet",
    "CLAUDE_CODE_AUTO_COMPACT_WINDOW": "200000"
  }
}
```

- `CLAUDE_CODE_SUBAGENT_MODEL` is the default for any dispatch that omits `model`
  (explicit per-dispatch models still win).
- `CLAUDE_CODE_AUTO_COMPACT_WINDOW` makes long sessions on 1M-context models
  compact at ~200k instead of growing to 500k+; every request re-reads the whole
  context, so this caps the cost per request. Trade-off: more compactions.
- Habit: `/clear` between unrelated tasks and between plan phases; the SDD
  ledger lets you resume a plan with "resume <plan path>".

## Install: Cursor

Cursor reads `.cursor-plugin/plugin.json` (skills + `hooks/hooks-cursor.json`).
Install it as a local plugin from this folder (Cursor's plugin settings /
"add local plugin"), and remove the upstream Superpowers plugin.

## Install: OpenCode

Add the folder (absolute path, `~` is not expanded) to `opencode.json`:

```json
{ "plugin": ["C:/Users/<you>/plugins/superpowers-lean"] }
```

The plugin injects the same compact bootstrap and registers the skills; it
already skips child (subagent) sessions.

## Other harnesses

Codex, Gemini, Pi, Hermes, Antigravity, Devin, Kimi and Muse manifests were
dropped from this fork. Use upstream for those, or copy `skills/` into the
harness's skills directory.
