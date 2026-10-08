# Decision record — shuttle-state

## Goal

Shuttle reads what an agent session is doing from what Claude Code itself reports, and the stops built on today's reading stop misfiring.
Four parts, one task:

1. A session-state model (`busy`, `idle-done`, `idle-stalled`, `asking`, `dead`, plus `unknown`) computed from hook events in shadow mode, logged beside today's guesses, read by no consumer.
2. A held turn end shows as a wait, like the gate and background-shell waits.
3. A turn end waiting on live background work is a wait, never a held stop or a question (#429); a Master's stuck outcome is never reported as a question.
4. A deliberate permission setup: lyx's own denies declared in one place, and a new deny of `python`/`python3` in Bash for every lyx-spawned agent.

## Scope

In:

- New hooks in the settings.json `claudeengine` writes: `UserPromptSubmit`, `StopFailure`, `Notification`, `SessionEnd`, all appending to the run's existing events file, every hook line stamped with a hook-side UTC time.
- A provider-neutral session-state reducer in `internal/shuttleengine`, its Claude signal parsing and liveness in `internal/shuttleengine/claudeengine`.
- Shadow logging: shuttle's wait loop, the orch watcher and batten's activity reading each log the state beside their own judgment; none decides on it.
- A read-only CLI verb `lyx shuttle state` printing the state and its change history for the live runs of the current worktree.
- The hold wait mark, the background-wait rules for #429, a regression pin for the stuck-outcome report.
- The `python` deny, the deny table, the `PATTERN-no-denied-recovery` list update.
- Replay-corpus cases and an `llm`-tier scenario for the new hook payloads.
- Docs in the same commit: `internal/shuttleengine/doc.go`, `internal/shuttleengine/claudeengine/doc.go`, `internal/shuttlecli`'s module doc, `pattern/PATTERN-no-denied-recovery.md`.

Out, per the board entry's own step plan (each consumer is its own later task):

- Any consumer acting on the state: batten's quiet notice, the orch watcher, creel, reed.
- The bounded resume of `idle-stalled` (continue, retry cap, backoff, visible halt): it acts on the state, so it lands with the first consumer.
- The operator's user-level Claude Code settings: lyx never reads or writes `~/.claude/settings.json`.
- The step-level watchdog (`verify-watchdog`), the Windows process-tree kill, `run-recovery`'s dead-process handling.
- Webster's re-step reclaim of a live Master (#429's last paragraph): with part 3 the wait no longer halts, so no re-step is triggered by it.

## Decisions

### D1 — Hook set and line format

Decision: `buildSettings` installs, beside the existing Stop and PreToolUse hooks, `UserPromptSubmit`, `StopFailure`, `Notification` and `SessionEnd` hooks, each appending its payload to the run's events file.
Every hook line, Stop and the AskUserQuestion record included, is written as one line in one write, with a top-level field `lyx_at` holding the hook's UTC time (RFC 3339, second resolution) inserted as the object's first field, the payload's own fields otherwise unchanged.
The hook command is plain POSIX shell runnable under git-bash, like today's (`date -u`, `cat`, `printf`), with no `jq` and no `lyx` binary.
A line without `lyx_at` (older runs, existing corpus) reads with the time the reader first saw it.
Rationale: hooks are Claude Code's own report of turn starts, API-error turn ends, waits on a permission and session exit; payloads carry no timestamp, and a hook-side stamp is the only time that does not depend on a poller being alive.
One write per line keeps concurrent hooks from interleaving; `ParseEvents` skips unknown `hook_event_name` lines and decodes fields into a map, so existing readers are unaffected.

### D2 — Signals are a separate parse, `ParseEvents` unchanged

Decision: the engine gets an optional capability, a session-signal parser, returning provider-neutral signals with their time: turn start, turn end (with its outstanding background tasks and their reporting signal), API-error turn end (with the error text), ask (a live AskUserQuestion, or a `Notification` whose type is a permission prompt or an elicitation dialog), idle notice (`Notification` `idle_prompt`), session end (with its reason).
`ParseEvents`, `EventKind` and the wait loop's classification take none of the new lines.
Rationale: shadow mode must change no behavior, and a new `EventKind` in the batch the wait loop classifies by its last event would.
The Claude payload knowledge stays in `claudeengine` (`PATTERN-shuttle-provider-seam`).

### D3 — The states and how they are derived

Decision: one pure reducer in `shuttleengine` folds a session's signals, in file order, plus three facts read at call time, into a state, its cause, the time it began, and the list of state changes:

- `dead`: the provider process is gone (D4), or a session-end signal is the newest signal. Cause: `process-gone` or the session-end reason.
- `busy`: the newest signal is a turn start; or a turn end with outstanding background work (cause `background`, listing the tasks).
  An interrupt marker in the transcript after the newest turn start makes it `idle-stalled` instead.
- `idle-done`: the newest signal is a turn end with no outstanding work and every output file present; a session with no output contract (the orch) reads `idle-done` at every such turn end.
  Output files are consulted before any negative state (`PATTERN-completion-signal`).
- `idle-stalled`: the newest signal is an API-error turn end (cause `api-error`, its text); an interrupt after the newest turn start (cause `interrupt`); or, in an autonomous run, a turn end with no outstanding work and an output file missing (cause `no-output`).
  An API-error turn end is the `StopFailure` signal, or a turn end whose transcript entry carries Claude Code's API-error marker (the existing `ActivityReader` reading).
- `asking`: the newest signal is an ask signal; or, in an interactive run, a turn end with no outstanding work and an output file missing.
- `unknown`: no signal yet past the prompt offset, an unreadable events file, liveness unprovable (D4), or a contradiction: an idle notice while the newest signal is a turn start. Cause names which.

The idle notice changes no state except through that contradiction.
A state change is recorded only when the state or its cause changes, stamped with the signal's `lyx_at`, or, for `dead` and `unknown` from a call-time fact, with the time of the reading.
Rationale: every state rests on a hook, the process or a marked transcript entry, never on elapsed time, mtime or pane text; an autonomous agent's question goes to its parent through the existing held notice, so without an explicit ask signal a missing-output turn end is `idle-stalled`, and only an interactive run has an operator waiting at the pane.
Telling "asked" from "stalled" by the message text would be a guess, which the board rules out.

### D4 — Liveness from the provider process

Decision: liveness is the Claude process of the session: the engine finds it through Claude's session registry entry for the session id (pid and `procStart`, the lookup `resume.go` already does) and reads it alive when the pid is alive with a matching start time.
No registry entry, an unreadable start time or a reused pid is `unknown` (cause `liveness-unproven`), not `dead`; a dead pid is `dead`.
The pane is not consulted by the reducer.
Rationale: the board's "liveness is the process"; reed's pane liveness is the wait loop's existing cross-check and stays where it is.

### D5 — Shadow logging at today's guessing sites

Decision: three sites log the state and never branch on it:

- Shuttle's wait loop folds new signals on each poll it already does and logs every state change at Info (`shuttle: session state`), with state, cause, since, and its own current classification (running, waiting, held, done, died).
  When the two disagree under a fixed mapping (waiting↔`busy`, held↔`idle-stalled`/`asking`, done↔`idle-done`, died↔`dead`, running↔`busy`), it logs `shuttle: session state disagrees` at Warn, once per disagreement.
- The orch watcher, on each idle probe it already makes, reads the orch session's state through a new `Runner` method and logs at Warn when the probe's idle answer disagrees with it (idle vs `busy`, busy vs an idle state).
- Batten's activity reading (`battencli`'s `readAgentActivity`) reads each live run's state beside its activity reading and logs it at Info when it would send a quiet or API-error notice, so each notice can be checked against the state.

Rationale: shadow mode's purpose is reading disagreements against real runs; logging where each guess is made pairs every guess with the state at that moment.
Bound: these sites only log; a failure to read the state is logged and changes no decision, notice or return.

### D6 — `lyx shuttle state`

Decision: a read-only verb under the existing `shuttle` cobra subtree, `lyx shuttle state`, printing JSON for every live run under the current worktree's run-directory root (live as `ReadAgentActivity` defines it): strand name, state, cause, since, and the change history; `--strand <name>` narrows to one.
It is wired through `cliwire` like its siblings, reads files only, writes nothing, and is exercised by the sandbox suite.
Rationale: the operator and the hub need to see a run's state without grepping logs while shadow mode is being judged; it is also the reducer's public test surface.

### D7 — A held turn end shows as a wait

Decision: while the wait loop holds a turn end, it shows a third wait label, `held`, through the same `wait.yaml` marker and `ReedOps.SetWaitMark` pane mark as the gate and background-shell waits, started at the held turn end and cleared at the next event, at any finalize and at Wait's return.
`lyx loom status`'s waiting line shows it like the others.
Batten's `waitLive` counts only gate and background-shell waits: a `held` mark does not suppress batten's quiet notice.
Rationale: run-recovery made every wait visible, and a held session without a mark looks like a hang; a held session is exactly what the quiet notice exists for, so it must not hide it.
Bound: the mark stays display and status only (`PATTERN-wait-mark-display-only`); the only reader that branches is batten's `waitLive`, which this decision narrows, never widens.

### D8 — A turn end waiting on live background work is a wait (#429)

Decision: in the wait loop, a turn end whose outstanding work is reported by the Stop payload (`SignalPayload`) keeps the run waiting for as long as each new turn end lists it running, bounded only by the run's own Timeout and the liveness check; it is never counted as a turn end, never held and never notified to the parent.
`background_shell_wait_min` applies only to shells reported by the transcript fallback (`SignalTranscript`), where today's expiry rules hold unchanged.
At a turn end where every output file exists and the only outstanding work is payload-reported shells not matching `Spec.AwaitedShellPrefixes`, the run finishes Done at once instead of waiting out `background_shell_wait_min`; those shells are logged and returned as today's `Result.ExpiredShells`, so webster's friction note still records them.
Outstanding forks and awaited-prefix shells keep the run waiting as today.
Rationale: since `shuttle-turns` the payload is authoritative, so a payload-reported running shell is live work, and a long measurement or simulation must never read as a stall; Claude Code wakes the session itself when the shell ends.
The transcript fallback is inference, so it keeps its bound.
With outputs present the agent's step is done, and a leftover shell is not a reason to delay the gate (the `shuttle-turns` PR-Gate finding).
Bound: a payload-reported shell that never ends now holds the run until its Timeout (for Master, `master_timeout_min`) instead of 10 minutes; the `background shells` wait mark shows it the whole time, and `verify-watchdog` owns a step that never returns.

### D9 — A stuck outcome is never reported as a question

Decision: no code change beyond a regression pin.
`shuttle-turns` removed the `shedadapters` asking branch that produced `webster master is asking a question`, and a Master that writes `outcome.yaml` with `outcome: stuck` plus `summary.md` now reaches shuttle's Done and webster's stuck result through `mapMasterDone`, with the verify gate passing on a non-done outcome.
The pin: such a Master run, through the webster shed adapter, yields a Stuck step whose reason carries the outcome's `stuck_reason`, and nothing in the result, the halt or the notice text calls it a question; under D3 the session reads `idle-done`.
Rationale: the card-batching run predates `shuttle-turns` (its batchifier commit landed at 15:44 on 2026-10-07, `shuttle-turns` at 18:23); the bug's path is gone, so the task pins it rather than rebuilding it.

### D10 — One declared table of lyx's denies

Decision: `claudeengine/settings.go` declares lyx's standing denies in one table, each row naming its tool matcher, its match, its steer reason, its system-prompt notice (or none, as for the webster fork guard today) and the config and run inputs that install it; `buildSettings` and `buildDenyNotice` both iterate the table.
The existing denies (Agent, Agent non-fork, AskUserQuestion, webster fork guard) move into it unchanged in behavior.
Rationale: the board's "deliberate permission setup"; the hooks and the notice already must not drift, and a table makes adding a deny one row.
lyx's agents run in bypass mode, so lyx needs no allow list; denies set in the operator's own settings stay the operator's, and `PATTERN-no-denied-recovery` keeps lyx's texts from prescribing them.

### D11 — Deny `python` in Bash

Decision: a new table row denies a Bash call whose command runs `python`, `python3` or `python3.<minor>` in command position: at the start, or after `;`, `&`, `|`, `(`, `$(`, a backtick, `env`, `exec`, `xargs`, `sudo` or `time`, optionally path-prefixed (`/usr/bin/python3`).
A `python` word as an argument (`grep python file`, a path containing it) is not matched.
Its steer: lyx agents never run Python; edit files with Edit or Write, search and read with grep, awk or cat.
It is announced in the deny notice and installed for every lyx-spawned session, interactive and autonomous, forks included, under a new `shuttle.yaml` key `claude_deny_python`, default `true`.
`cmd/lyx/norecoverydeny_test.go`'s denied-form list gains `python` and `python3`, and `pattern/PATTERN-no-denied-recovery.md` names this task's table as the owner of the real set.
Rationale: the operator's rule after the `board-model` 75-minute stdin hang and the `agents-no-scripted-edits` script edit.
Bound: it can block a legitimate Python run in a Python target repo, which sets `claude_deny_python: false`; verify commands run through `verifytree` in Go, not the agent's Bash tool, so the deny never touches them.

## Constraints

- `PATTERN-shuttle-provider-seam`: hook names, payload fields, the registry and the transcript markers live only in `claudeengine`; the states, the reducer and the signal types are provider-neutral in `shuttleengine`.
- `PATTERN-completion-signal`: every negative state consults the output files first; the reducer adds no done-when writer or reader.
- `PATTERN-wait-mark-display-only`: the `held` pane mark goes through `SetWaitMark` from `shuttleengine` only; no Go decision reads pane options.
- `PATTERN-told-geometry`, `PATTERN-cwd-resolution`, `PATTERN-lyxdirs-single-declarer`: the reducer and readers are handed run directories; only `shuttlecli` resolves the worktree, through `cliwire` (`PATTERN-cliwire-sole-wiring`, `PATTERN-cli-cobra`).
- `PATTERN-orch-pane-single-writer`: the orch watcher's shadow log types nothing.
- `PATTERN-no-denied-recovery`: the denied list grows by `python`/`python3`; no stencil or spec names them as a step.
- `PATTERN-sandbox-coverage`: the sandbox suite exercises `lyx shuttle state`.
- `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-test-economy`: the reducer and parse tests are untagged with an injected clock; hook-payload capture is `llm` tier through `llmkit`; corpus cases extend the existing table test.
- `PATTERN-spawn-observability`: no new process spawn; the hook commands run inside Claude Code.
- `PATTERN-comment-line-breaks` and semantic line breaks in every doc touched.
- Hook commands run under git-bash on Windows: POSIX shell, `date`, `cat`, `printf` only.

## Auto-mode assumptions

- The bounded resume of `idle-stalled` is out of scope: the board's own plan says nothing changes behavior until the state is proven, and resume is a consumer.
- An autonomous turn end without output and without an ask signal is `idle-stalled`, not `asking`.
- Liveness comes from Claude's session registry; a missing entry is `unknown`, not `dead`.
- `lyx shuttle state` is added as a read-only verb for shadow-mode inspection.
- The python deny is on by default with a per-repo off switch, `claude_deny_python`.
- Payload-reported background shells wait to the run Timeout; `background_shell_wait_min` survives only for transcript-reported shells.
- A done-at-turn-end with only leftover payload-reported shells finishes at once.
- The stuck-outcome bug is fixed by `shuttle-turns`; this task pins it.
- `StopFailure`'s payload fields (`error`, `error_details`, `last_assistant_message`) are taken from Claude Code 2.1.294's own hook schema; no real `StopFailure` payload has been captured yet.

## Open risks

- `StopFailure` and `Notification` payload shapes are unverified against a captured run; a field that differs degrades that signal, and the state falls back to the transcript API-error marker or `unknown`.
- Whether `UserPromptSubmit` fires for a turn Claude Code starts on its own (a background task's completion, a queued notification) is unverified; if it does not, such a turn reads `busy` from the preceding waiting turn end, which is correct, but a turn after an `idle-done` that is woken this way reads `idle-done` until its Stop.
- The `idle_prompt` notice fires after Claude Code's idle threshold; an older Claude Code that fired it while background agents ran (fixed upstream) would log false contradictions.
- D8 lets a hung payload-reported shell hold a Master until `master_timeout_min`.
- The python matcher is a regex over the JSON payload; a command quoting `python` in command position inside a string argument is a false deny, and an unusual invocation (`py`, `uv run`) is a miss.

## Acceptance criteria

- A session's events file gains one line per `UserPromptSubmit`, `StopFailure`, `Notification` and `SessionEnd` hook, each with `lyx_at`, and the existing Stop and ask lines keep parsing exactly as before.
- Every existing corpus case replays through `ParseEvents` and the wait loop with unchanged readings, and each case also states the session state its events reach.
- The reducer, over replayed signals, reads: a turn start → `busy`; a waiting turn end → `busy` with its tasks; a turn end with outputs → `idle-done`; an orch-style session's turn end → `idle-done`; an autonomous no-output turn end → `idle-stalled` `no-output`; an interactive one → `asking`; an AskUserQuestion or permission notification → `asking`; a `StopFailure` → `idle-stalled` `api-error` with its text; an API-error transcript entry without `StopFailure` → the same; an interrupt marker after a turn start → `idle-stalled` `interrupt`; a dead registry pid → `dead`; a session end → `dead`; no registry entry → `unknown`; an idle notice after a turn start → `unknown` contradiction; no signals → `unknown`.
- The change history holds one entry per state-or-cause change with the hook's time, and none for a repeated identical signal.
- The `fork-killed-api-error` corpus case reads `idle-stalled` `api-error` at its API-error turn end.
- The wait loop logs each state change and logs a disagreement once when its own classification and the state differ under D5's mapping; no wait-loop verdict, notice or return changes because of the state.
- The orch watcher and batten log the state beside their judgment, and a state read failure changes none of their decisions.
- `lyx shuttle state` prints each live run's state, cause, since and history as JSON, `--strand` narrows it, and it writes nothing.
- A held turn end writes `wait.yaml` and sets the pane mark with label `held`, both cleared at the next event and at return; `lyx loom status` shows it; batten's quiet notice still fires for a held run.
- A turn end listing a payload-reported running shell keeps an autonomous run waiting past `background_shell_wait_min` with no held notice, and the run continues at the agent's next turn end.
- A transcript-reported shell still expires after `background_shell_wait_min` as today.
- A turn end with every output file present and only payload-reported unawaited shells outstanding finishes Done at once and names those shells in `ExpiredShells`; with an outstanding fork or awaited-prefix shell it keeps waiting.
- A webster Master run that writes `outcome: stuck` and `summary.md` yields a Stuck step carrying the stuck reason, and no result, halt note or notice text calls it a question.
- A Bash call running `python3 -c …`, `cd x && python script.py`, `/usr/bin/python3 - <<EOF` or `xargs python3` is denied with the steer; `grep python file` and `ls python-tools/` are not; `claude_deny_python: false` installs no python deny; the deny notice announces it exactly when it is installed.
- The deny table drives both `buildSettings` and `buildDenyNotice`, and the existing denies' hooks and notices are byte-identical to today's.
- `norecoverydeny_test.go` fails on a stencil that offers `python3` as a step.
- An `llm`-tier scenario captures real `UserPromptSubmit`, `Notification` and `SessionEnd` payloads, and its events reduce to the states the run went through.

## Notes for the plan writer

- `internal/shuttleengine/activity.go`'s `ReadAgentActivity` and `runIsLive` are the model for a files-only reader over a run-directory root; the API-error transcript reading is `claudeengine`'s `ActivityReader`.
- Claude's session registry lookup is in `claudeengine/resume.go` (`registryEntry`, `checkResumable`).
- The hook commands and the existing deny/notice drift guard are in `claudeengine/settings.go` (`buildSettings`, `denyInstalls`, `buildDenyNotice`, the steer-character panic check).
- Claude Code 2.1.294's hook names include `StopFailure` (fields `error`, `error_details`, `last_assistant_message`), `Notification` (`notification_type` such as `idle_prompt`, `permission_prompt`), `UserPromptSubmit` and `SessionEnd`; the Stop payload also carries `session_crons`.
- The background-shell expiry and `ExpiredShells` live in `internal/shuttleengine/wait.go`; webster's friction note on expired shells is in `internal/websterengine/runlevel.go`.
- Batten's `waitLive` comes from `battencli/wire.go`'s `readAgentActivity` and is read in `battenshed/notice.go`.
- The orch watcher's idle probe is `orchengine.Watcher.probeIdle`; the orch session has an events file but no wait loop, so its state is read through a `Runner` method beside `SessionIdle`.
