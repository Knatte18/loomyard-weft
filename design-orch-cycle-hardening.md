# Orch context cycling and long-run hardening

`orch-cycle-hardening` · Task · bug, enhancement, orch, config, batten · running · PR-Gate

# Goal

`lyx orch` cycles its context correctly and runs for days without hand repair.
The stale reading after `/compact` and the clear-or-compact choice live in the same watcher code, and the hardening list touches the same adopt and context-reading paths.

# Decisions from the task discussion (operator, 2026-10-05)

- **Cycle mode.** `cycle_mode: clear|compact` in `orch.yaml`, default `compact`.
  A compact cycle keeps the session and its Remote Control link and passes the handoff instruction as `/compact`'s focus argument, so it writes no handoff file.
- **`permission_mode: prompt` is removed.** The orch never prompts for each action; `bypass` is the only mode, and the config key, its template comment and the orchengine doc say so.
- **Watchers.** `orch-no-watcher` is part of this task: the orch relies on driver and batten messages, and any gap they leave is closed in lyx code, never by a scratch watcher.
  Background shells and Monitors die with the process, which matters at adopt and reboot (a new `claude --resume` process), not at compact; Discussion verifies whether they survive `/clear`.


## Orch watcher cycles on a stale reading after /compact

From note `orch-compact-stale-reading`.

On 2026-10-04 the operator ran `/compact` in the orch session at 10:09:49Z.
It finished at 10:11:15Z (`compact_boundary`, trigger manual, preTokens 354815, postTokens 13673).
At 10:14:53Z the watcher sent the soft-cycle stencil and then cleared and resumed the session from a handoff, although the context held about 14k tokens.
`state.json` afterwards shows `cycle_trigger: soft`.

Cause: the watcher updates `LastContextTokens` only at a turn end (`internal/orchengine/watcher.go`, the `ContextTokens(ev)` call per turn-end event).
A `/compact` produces no turn end, so the reading stayed at the pre-compaction 354815, above `soft_threshold_tokens` (300000), and the soft trigger fired once `soft_idle_s` had passed.
`transcriptContextTokens` in `internal/shuttleengine/claudeengine/usage.go` would return the same stale number even if re-read, since it takes the last main-chain assistant entry's usage, which is still the pre-compaction one.

Expected: a manual compaction resets the reading, so the operator's `/compact` is not followed by a cycle.
Options: treat a `compact_boundary` entry newer than the last assistant entry as the reading (its `postTokens`), and re-read the transcript before firing a soft or hard trigger rather than trusting the stored turn-end reading.
The same applies to Claude Code's own auto-compaction (a residual named in `internal/orchengine/doc.go`).


From GitHub #356 (intake 2026-10-04).

## Orch cycle mode as config: clear or compact

From note `orch-cycle-compact`.

Seen 2026-10-03: the orch's context cycle uses `/clear` plus a handoff file, which starts a new conversation and may drop a Remote Control link the operator relies on from a phone.
Cycling through `/compact` keeps the same session, so Remote Control and session identity survive.
Idea (operator): the cycle mode as a config option, `clear` or `compact`; weigh what the handoff file gives that a compaction summary does not.

## lyx orch runs for days without hand repair

From note `orch-live-hardening`.

### Goal

The hub runs under `lyx orch` for days without hand repair: an adopt keeps the hub's watchers, the orch pane keeps its height, permission modes mean what they say, and a long transcript stays cheap to read.

### From the PR #337 review (2026-10-02)

- **Watchers after adopt.** The hub's background shells and Monitors die with the session it exits; `orch-template-adopt.md` does not tell the resumed session they are gone or to re-arm them.
- **Window size on attach.** With `window-size latest`, a `reed attach` from VS Code resizes the orch window to that client (seen on the PR #342 review, after #340's folderOpen fix landed).
- **`permission_mode: prompt`** adds no flag, so the session falls back to the user's `defaultMode` (`auto` here); the template comment and the orchengine doc promise prompts.
- **Smoke bypass check** proves nothing where `~/.claude/settings.json` allows `Bash`; assert the bypass footer or use a tool outside the allow list.
- **Large-session adopt** is untested: startup within `startup_timeout_s` and any resume dialog on a real hub transcript (decision record, open risk 2).
- **Stale context reading.** `ResetForFreshLaunch` keeps `LastContextTokens`, so `status` shows the previous session's reading until the first turn end after an adopt.
- **PID reuse** can refuse an adopt falsely; the registry's `procStart` tells a reused pid apart.
- **Transcript read cost.** `ContextTokens` reads the whole transcript on every idle turn end; read from the end.

## The orch relies on driver messages, not a scratch watcher

From note `orch-no-watcher`.

Seen 2026-10-03: the hub orch watches each driven run with a hand-written `.scratch/hub/watch.sh`, a background shell loop that polls the run's `status.json` every 30 s and exits on the first non-running state change.
It lives outside lyx, untested and uncommitted, and has to be re-armed by hand after every context cycle.
The driver already messages its parent (`ly:orch`) at every stop it cannot handle, which is the path the orch should rely on.
Idea (operator): question whether the orch needs a watcher at all; if a gap remains that the driver's message does not cover (a driver that dies without messaging, a stop the driver does not report), close it in lyx's own code, never with a scratch script.
Likely home: `lyx batten run`, whose Run-Shed row already polls the same `status.json` every 30 s.
Batten could message its parent on a non-running state change, and check the driver strand's liveness through reed to cover the case `internal/battenshed/doc.go` leaves out by design (a driver that dies, parks on a prompt, or stops with the run non-terminal).
Batten can also be started by hand from a terminal with no orch above it, so the parent is optional: message it when one is recorded, otherwise only log.
