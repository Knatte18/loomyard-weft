# shuttle: read session state from hooks in shadow mode, fix #429

Shuttle now computes what an agent session is doing from the hook events Claude Code reports, and logs it beside today's guesses.
No consumer acts on the state yet.
The same change stops a turn end that waits on live background work from reading as a held stop or a question, and adds a standing `python` deny for lyx-spawned agents.

Closes #429

## What changes

- **Session state (shadow mode).**
  A provider-neutral reducer in `shuttleengine` folds session signals into `busy`, `idle-done`, `idle-stalled`, `asking`, `dead` or `unknown`, with a cause, a `since` time and a change history.
  Claude payload parsing, registry liveness and the transcript interrupt marker stay in `claudeengine`.
- **New hooks.**
  `buildSettings` installs `UserPromptSubmit`, `StopFailure`, `Notification` and `SessionEnd`, appending to the run's existing events file.
  Every recording hook, Stop and the ask record included, first appends a stamp line with a hook-side UTC time, then the payload exactly as today.
  `ParseEvents` skips stamp lines, so the wait loop's input is unchanged.
- **Shadow logging.**
  The wait loop logs each state change and, once per disagreement, `shuttle: session state disagrees`.
  The orch watcher logs against its idle probe, and batten logs beside a quiet or API-error notice.
  A failure to read the state changes no decision.
- **`lyx shuttle state`.**
  A read-only verb printing JSON for every running record of the current worktree, with `--strand` to narrow.
- **Held turn end shows as a wait.**
  The wait loop sets a `held` wait mark through the existing `wait.yaml` and pane mark; `lyx loom status` shows it.
  Batten's `waitLive` ignores it, so the quiet notice still fires for a held run.
- **#429, background work.**
  A turn end whose Stop payload reports running background tasks keeps the run waiting up to the run's own Timeout.
  `background_shell_wait_min` now applies only to shells the transcript fallback reports.
  In a gated run with every output present and only unawaited payload-reported shells left, the run finishes Done at once and returns them in `ExpiredShells`.
  A Master's stuck outcome is pinned by a regression test as a Stuck step, never a question.
- **Deny table and `python` deny.**
  Lyx's denies are declared in one table in `claudeengine/settings.go` that drives both the hooks and the deny notice; existing denies are byte-identical.
  A new row denies Bash `python`, `python3` and `python3.<minor>` in command position, switched by the `shuttle.yaml` key `claude_deny_python` (default `true`).
  `PATTERN-no-denied-recovery` and `norecoverydeny_test.go` name the new entries.
- **Batten activity.**
  `ReadAgentActivity` takes the events file's activity from the newest line `ParseEvents` parses, not the file's mtime, so the new hook and stamp lines (an `idle_prompt` append in particular) do not restart the quiet clock.

## Design decisions to review

- The reducer orders signals by file order and reads `lyx_at` only for `since`, so no state depends on comparing a hook stamp with a transcript timestamp.
- Precedence puts process facts (`dead`, `unknown` from unproven liveness) above output-derived states, which is safe only while nothing acts on the state.
  The first consumer that acts on `dead` must apply the `PATTERN-completion-signal` output check first.
- A missing registry entry reads `unknown` `liveness-unproven`, never `dead`.
- An autonomous turn end without output and without an ask signal is `idle-stalled`, not `asking`; only interactive runs read `asking` there.
- The python deny is a guardrail, not a barrier: it greps the raw PreToolUse payload, lets through `bash -c`, `uv run` and similar, and can falsely deny `python` after a separator inside a quoted argument.
  A Python target repo sets `claude_deny_python: false`.
- A payload-reported shell that never ends now holds a Master until `master_timeout_min`, then ends `OutcomeTimeout`.
  A log line, `shuttle: background shell past wait min`, marks each such shell once.
- History entries resting on a reading-time check (output files, transcript markers, transcript-fallback tasks) are per reader by design.
- The bounded resume of `idle-stalled` and every consumer of the state are out of scope.

## To check by hand

- The `llm`-tier scenario capturing real `UserPromptSubmit`, `Notification` (`idle_prompt`) and `SessionEnd` payloads compiles under `go vet -tags llm` but was not run.
- No real `StopFailure` payload has been captured; its fields come from Claude Code 2.1.294's hook schema.
- Unverified against a live run: whether Claude Code removes its session-registry entry at a normal exit (so exits mostly read `unknown`), whether `UserPromptSubmit` fires for turns Claude Code starts itself, and whether `idle_prompt` repeats within one idle stretch.
- `internal/shuttlecli/modelpin_test.go` changed only because the spawn-site count it guards rose by one with the new `llm` scenario.
- New appenders to the events file can hit the existing gap between a payload's write and its newline; a Stop line lost that way holds the run to its Timeout, as today.
