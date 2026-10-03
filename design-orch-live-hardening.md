# lyx orch runs for days without hand repair

`orch-live-hardening` · Someday · feature

## Goal

The hub runs under `lyx orch` for days without hand repair: an adopt keeps the hub's watchers, the orch pane keeps its height, permission modes mean what they say, and a long transcript stays cheap to read.

## From the PR #337 review (2026-10-02)

- **Watchers after adopt.** The hub's background shells and Monitors die with the session it exits; `orch-template-adopt.md` does not tell the resumed session they are gone or to re-arm them.
- **Window size on attach.** With `window-size latest`, a `reed attach` from VS Code resizes the orch window to that client (seen on the PR #342 review, after #340's folderOpen fix landed).
- **`permission_mode: prompt`** adds no flag, so the session falls back to the user's `defaultMode` (`auto` here); the template comment and the orchengine doc promise prompts.
- **Smoke bypass check** proves nothing where `~/.claude/settings.json` allows `Bash`; assert the bypass footer or use a tool outside the allow list.
- **Large-session adopt** is untested: startup within `startup_timeout_s` and any resume dialog on a real hub transcript (decision record, open risk 2).
- **Stale context reading.** `ResetForFreshLaunch` keeps `LastContextTokens`, so `status` shows the previous session's reading until the first turn end after an adopt.
- **PID reuse** can refuse an adopt falsely; the registry's `procStart` tells a reused pid apart.
- **Transcript read cost.** `ContextTokens` reads the whole transcript on every idle turn end; read from the end.

## Cycle by compact instead of clear (operator idea, 2026-10-03)

The orch's context cycle uses `/clear` plus a handoff file, which starts a new conversation and may drop a Remote Control link the operator relies on from a phone.
Cycling through `/compact` keeps the same session, so Remote Control and session identity survive.
The operator wants the cycle mode as a config option (`clear` or `compact`), not a fixed switch; weigh what the handoff file gives that a compaction summary does not.


## Reopening the prime's VS Code window (seen 2026-10-03)

- A snap auto-refresh of VS Code (rev 257 → 267 at 12:44) left the old instance running; batten's next `code <pair>` call then started a second instance, which restored the prime as a bare folder without the hub workspace's title and colour, and the operator had to close windows by hand.
- Reopening the prime correctly takes `lyx ide spawn <prime-name>`, whose folderOpen chain (`reed up`, `reed add --unless-name orch`, `reed attach`) reattaches to the orch without starting a second Claude; a plain `code` on the prime gets the stale mill-managed `.vscode/tasks.json` and attaches nothing.
- Want: `lyx ide spawn` with no argument opens the worktree it is run in, so the operator restores the prime with one short command; run from the hub root or any hub directory outside a worktree (today it fails there, and the operator had to cd into the prime), it opens the prime.
- Want: a `lyx ide open [<slug>]` that only launches VS Code and writes nothing: the prime through its existing hub workspace file, a pair through its folder and the `tasks.json` already there. `lyx ide spawn` stays the configuring verb; today it rewrites a driven pair's attach-only chain with the interactive one, whose `reed add claude` can start a second Claude in that pair. The no-argument and hub-root defaults above apply to `open`.
- Idea, undecided (operator, 2026-10-03): keep the prime's hub workspace as the one window, and have fabric add each pair to its `folders` on `add` and drop it on `remove`/teardown; VS Code picks up workspace-file edits live. The operator rarely reads a pair's files (everything goes through the orch) but does look at plan files and similar now and then, which `_portals` (each pair's `_lyx/`) already covers and is kept for. Open points if pursued: per-folder `folderOpen` tasks would run in the prime window (and batten's own pair window would then go), a full pair copy multiplies indexing and search unless only `_lyx/` is added or excludes are set, and the prime must stay the first folder or the extension host restarts.
