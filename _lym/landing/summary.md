# Codex launches carry Claude's standing items and user setup

A Codex session now gets, as developer instructions, the standing items a Claude session gets in its appended system prompt, plus the user's Claude setup and subagent terms.
Before, those items sat in the user turn on Codex, below the developer level, and a Codex session saw none of the user's Claude setup.

## What changes

- **Developer instructions.**
  `codexengine` passes `developer_instructions` as one single-line TOML string, in this order:
  the items Claude's appended system prompt holds (subagent passage, deny notice, creel trust sentence, heddle passages);
  the user's instruction files; the instruction-file pointers; the skill listing; the plugin session-start contexts.
  The prompt file then holds only the spec's prompt.
  An empty set omits the key, since Codex refuses a blank value.
  `Prepare` refuses a launch whose argument reaches the 128 KiB `MAX_ARG_STRLEN` cap, naming its size, its largest item and the way forward.
- **Subagent terms.**
  `agents.default_subagent_reasoning_effort` carries the effort `agents.yaml` gives the role, and the subagent passage states Claude's rules in Codex's terms.
  `agents.max_concurrent_threads_per_session` carries the new `codex.yaml` key `max_subagent_threads` (template default 16).
- **User setup.**
  `claudeengine` resolves the user's global instruction file, each CLAUDE.md above the repository root, the skills the session scope lists with their SKILL.md paths, and the `SessionStart` command hooks of the plugins the scope turns on.
  `codexengine` receives it through a new `shuttleengine` seam wired in `shuttleengines`.
  A hook that fails or times out is left out with a warning.
  Each hook is bounded by its plugin's own timeout, else the new `codex.yaml` key `plugin_hook_timeout` (template default 10s).
  The hook runner and Codex's sandbox probe are injectable seams of `Prepare`.
- **Imports.**
  The new leaf package `internal/claudeimports` expands `@` imports by Claude Code's rule (relative to the importing file, five hops, never inside code), and `claudeengine` and heddle share it.
- **Instruction files.**
  `project_doc_max_bytes` is raised to the size of the files Codex reads from the root to its anchor, never below Codex's default.
  The `codex-file` render writes `AGENTS.override.md` carrying `AGENTS.md` and the import-expanded CLAUDE.md.
  Each render records the in-worktree imports it resolved, and heddle's event matcher re-renders the target when one changes.
  The developer instructions name each other CLAUDE.md that Codex does not read itself.
- **Heddle.**
  New shared prompt target `agents-prompt` renders into `claude-prompt` and `codex-prompt`; `lym heddle show agents-prompt` is refused like `agents-file`.
  The hub reconcile renders heddle's file targets into every live worktree, so a shipped-entry change reaches existing worktrees.

## Design decisions

- User-setup text goes per session, never into the shared Codex home, because a Claude config directory can differ per worktree.
- Claude's user-setup resolution stays in `claudeengine` and reaches `codexengine` only through the `shuttleengine` seam.
- When the Claude config directory cannot be resolved, `UserSetup` drops only what needs it (the global file, plugin skills and plugin contexts), keeps project skills and CLAUDE.md files above the root, and adds one warning naming `CLAUDE_CONFIG_DIR` or `HOME`.
- Hooks run with `CLAUDE_PLUGIN_ROOT` and with `CLAUDE_PROJECT_DIR` set to the session's working directory, which can be a subdirectory of the worktree.
  `session_id` is the lym run's id, since no Codex session id exists yet.
- An import that resolves outside the worktree raises no worktree event, so an edit to it reaches Codex at the target's next render.
- A machine on an older lym refuses a store holding an `agents-prompt` entry until upgraded; no shipped entry uses the target.

## Check by hand

- Launch a Codex session in a worktree and confirm its developer instructions follow the stated order, with the user's global file, a plugin's session-start output and the skill listing present.
- Review finding `hook-timeout-row-load-bound-no-waitdelay`: the plugin hook timeout row's load bound sets no `WaitDelay` (`internal/shuttleengine/claudeengine/pluginhook.go`); confirm a hook that leaves a child holding its pipes cannot stall `Prepare` past the timeout.
- Review finding `overview-codex-launch-docs`: `docs/overview.md`'s Codex launch paragraph; confirm it matches the final behaviour.
- Review finding `prepare-workdir-unresolved-anchor`: `Prepare`'s working directory when the anchor does not resolve; confirm the hook's `cwd` and `CLAUDE_PROJECT_DIR` are right there.
- Review finding `unresolvable-config-dir-untested`: the unresolvable-config-dir path of `UserSetup` in `internal/shuttleengine/claudeengine/usersetup.go` has no test; confirm the degraded result by hand.
