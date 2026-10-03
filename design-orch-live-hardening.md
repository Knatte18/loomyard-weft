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
Cycling through `/compact` keeps the same session, so Remote Control and session identity survive; weigh what the handoff file gives that a compaction summary does not.
