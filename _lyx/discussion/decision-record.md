# Decision record — shuttle-state

## Goal

Shuttle reads what an agent session is doing from what Claude Code itself reports, and the stops built on today's reading stop misfiring.
Four parts, one task:

1. A session-state model (`busy`, `idle-done`, `idle-stalled`, `asking`, `dead`, plus `unknown`) computed from hook events in shadow mode, logged beside today's guesses, read by no consumer.
2. A held turn end shows as a wait, like the gate and background-shell waits.
3. A turn end waiting on background work the Stop payload reports running is a wait, never a held stop or a question (#429); a Master's stuck outcome is never reported as a question.
   Shells only the transcript fallback reports keep today's expiry and held stop.
4. A deliberate permission setup: lyx's own denies declared in one place, and a new deny of `python`/`python3` in Bash for every lyx-spawned agent.

## Scope

In:

- New hooks in the settings.json `claudeengine` writes: `UserPromptSubmit`, `StopFailure`, `Notification`, `SessionEnd`, all appending to the run's existing events file, every hook's payload line preceded by a stamp line carrying a hook-side UTC time.
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
Every hook, Stop and the AskUserQuestion record included, appends two lines: first a stamp line `{"lyx_stamp":"<hook event name>","lyx_at":"<time>"}` by one `printf`, the time from `date -u` (RFC 3339, second resolution) passed as a `%s` argument; then the payload by today's command, `cat >> … && printf '\n' >> …`, unchanged.
The two are joined by `;`, so a failed stamp never blocks the payload append, and the hook's exit status is the payload append's.
The hook command is plain POSIX shell runnable under git-bash, like today's (`date -u`, `cat`, `printf`), with no `jq` and no `lyx` binary.
A payload line takes its time from the nearest preceding stamp line that names its `hook_event_name` and that no other payload line has taken; `claudeengine`'s signal parser (D2) does that pairing.
An incremental reader (the wait loop, a polling cursor) carries every stamp line not yet paired with a payload into its next batch, so a poll landing between a stamp and its payload pairs them as a whole-file reading does.
A payload line with no such stamp, or whose stamp has an empty or malformed `lyx_at` (older runs, the existing corpus, a failed `date`), has no time: whatever state it causes carries an empty `since`, the same for every reader.
Rationale: hooks are Claude Code's own report of turn starts, API-error turn ends, waits on a permission and session exit; payloads carry no timestamp, and a hook-side stamp is the only time that does not depend on a poller being alive.
The stamp is a separate short append, so the payload's own write stays today's.
A stamp line has no `hook_event_name`, so `ParseEvents` skips it, as it skips any line without one.
Bound: the Stop and ask record payload lines are byte-identical to today's and written by today's command, so `ParseEvents` yields the same events from them, `Raw` included, and the wait loop's input changes only by the stamp lines it skips.
The events file's mtime is the one other input that changes: `ReadAgentActivity` counts it toward `LastActivity`, which batten's quiet and API-error notices read.
Every new append moves it, and all but one land when the transcript is written too (a prompt, an API-error turn end, a session end, a stamp beside its payload); the exception is the `Notification` `idle_prompt` append, which lands after the session has sat idle for Claude Code's idle threshold.
So batten's quiet notice for an idle session fires later by at most that threshold, once per idle notice, which this task accepts in shadow mode and lists under Open risks; no other reader of the events file's mtime or size exists.
Today's gap between a payload's `write` and its newline's stays: a concurrent append landing there corrupts both lines, `ParseEvents` skips them as it skips a malformed line today, and a skipped Stop line leaves the run waiting to its Timeout.
The four new hooks are more appenders that can hit that gap; the gap is hit only when two hooks of one run's session append at once, such as a `Notification` beside another hook, which this task accepts as an Open risk.
Two hooks of the same name running at once can swap their stamps, mis-stamping each by the time between them.
The recording hooks (`UserPromptSubmit`, `StopFailure`, `Notification`, `SessionEnd`, Stop and the ask record) print nothing to standard output, so none adds to the model's context; the PreToolUse deny hooks (D10, D11) print their deny JSON as today.
The four new hooks end in `; true`, so they exit 0 whether or not the append succeeds and none blocks a prompt or a turn end; Stop and the ask record keep today's exit status, where a failed append exits 1, which shows as a hook error and blocks nothing, as the `cat >> … && printf` chain does today.
The events file, `lyx_at` and the transcript a payload names all sit where the agent can write, so the whole state is advisory in shadow mode, and no consumer acts on it yet.
The reducer still reads `lyx_at` only for `since`: it orders signals by file order and never by `lyx_at`, so no state or cause depends on a clock comparison between a hook stamp and a transcript timestamp, and no decision, notice or return depends on the state (D3, D5).

### D2 — Signals are a separate parse, `ParseEvents` unchanged

Decision: the engine gets an optional capability, a session-signal parser, returning provider-neutral signals with their time: turn start, turn end (with its outstanding background tasks and their reporting signal: the Stop payload, or the transcript fallback read at parse time), API-error turn end (with the error text), ask (a live AskUserQuestion, or a `Notification` whose type is a permission prompt or an elicitation dialog), idle notice (`Notification` `idle_prompt`), session end (with its reason).
`ParseEvents`, `EventKind` and the wait loop's classification take none of the new lines.
Rationale: shadow mode must change no behavior, and a new `EventKind` in the batch the wait loop classifies by its last event would.
The Claude payload knowledge stays in `claudeengine` (`PATTERN-shuttle-provider-seam`).

### D3 — The states and how they are derived

Decision: one pure reducer in `shuttleengine` folds a session's signals, in file order, plus a set of facts its caller reads and hands it, into a state, its cause, the time it began, and the list of state changes.
The fold is pure; reading the facts is the caller's, through the engine where the fact is provider-specific.
The facts, each with its source:

- Run mode (autonomous or interactive), the output-file list and the prompt offset: the run record (`run.json`'s `interactive`, `outputFiles` and `promptOffset`), for every reader.
  The orch session is a shuttle run with its own record, which the orch watcher reaches through the `Runner` as it reaches the orch's events today: interactive, with a never-written sentinel as its one output file.
  The reducer folds only signals past the prompt offset.
- Which output files exist: the caller stats them at reading time, and the fold applies that one result to every turn end it folds in that reading.
  The wait loop folds each poll's new signals with that poll's stat, so its turn ends are judged by the files as they were when it first read them; a files-only reader folds the whole file with its one stat.
  The wait loop keeps its reducer result between polls, and at a poll with no new turn end it re-judges only the newest turn end, when that reads `no-output` or interactive `asking`, against the poll's stat; earlier history entries stay as logged.
- The session id the liveness lookup uses: the `session_id` of the newest hook line past the prompt offset, else `run.json`'s `sessionId`, because a `/clear` or `/resume` changes the session id inside a running process while `run.json` keeps the spawn-time one.
- Provider-process liveness: `claudeengine` (D4).
- The API-error marker on a turn end's transcript entry: `claudeengine`'s existing `ActivityReader` reading of the Stop payload's `transcript_path`.
  It reports the transcript's newest turn-end entry, so it applies only to the newest turn end of a fold; an older turn end folded in the same reading takes no transcript marker.
- The interrupt marker: a new `claudeengine` read beside `ActivityReader`, over the transcript the newest turn start's `UserPromptSubmit` payload names through `transcript_path`, applied only to that newest turn start.
  It reports whether the transcript's newest main-chain entry is Claude Code's interrupt marker, a user entry whose text starts with `[Request interrupted by user`, with that entry's own timestamp; any later main-chain entry (a new prompt, an assistant entry) means no interrupt.
  "After the newest turn start" is transcript order alone, never a comparison with `lyx_at`; a missing `transcript_path` or an unreadable transcript reads as no interrupt.

The states:

- `dead`: the provider process is gone (D4), or the newest signal is a session end whose reason ends the process (`logout`, `prompt_input_exit`, `other`). Cause: `process-gone` or the session-end reason.
- `busy`: the newest signal is a turn start (cause `turn`); or a turn end with outstanding background work (cause `background`, listing the tasks).
  A turn start after a `busy` `turn` changes neither state nor cause, so it records no entry.
  An interrupt marker in the transcript after the newest turn start makes it `idle-stalled` instead.
- `idle-done`: the newest signal is a turn end with no outstanding work and every output file present; a session with an empty output list reads `idle-done` at every such turn end.
  The orch's sentinel is never written, so its turn ends read interactive `asking`: the operator is at its pane.
  Output files are consulted before either output-derived negative state, `no-output` or interactive `asking`.
  `dead` and `unknown` are process and reading facts ranked above them (rules 1 to 4 below), so a finished run whose process is gone reads `dead`; that is safe only because no consumer acts on the state, and the first consumer that acts on `dead` applies `PATTERN-completion-signal`'s output check before it does.
- `idle-stalled`: the newest signal is an API-error turn end (cause `api-error`, its text); an interrupt after the newest turn start (cause `interrupt`); or, in an autonomous run, a turn end with no outstanding work and an output file missing (cause `no-output`).
  An API-error turn end is the `StopFailure` signal, or a turn end whose transcript entry carries Claude Code's API-error marker (the existing `ActivityReader` reading).
- `asking`: the newest signal is an ask signal; or, in an interactive run, a turn end with no outstanding work and an output file missing.
- `unknown`: no signal yet past the prompt offset, an unreadable events file, liveness unprovable (D4), a contradiction (an idle notice while the newest signal is a turn start), or a session end whose reason is `clear` or `resume` as the newest signal, which ends a session inside a process that keeps running and holds until the next signal. Cause names which, the session-end reason for the last.

Where several conditions hold, the first in this order decides the state and its cause:

1. A process-ending session end as the newest signal: `dead`, cause the session-end reason, whether or not the process is still found.
   A `clear` or `resume` session end takes no part here; it decides only under rule 5, as `unknown` with its reason.
2. A dead provider process (D4): `dead` `process-gone`.
3. Liveness unproven (D4): `unknown` `liveness-unproven`, also over a turn end with every output present, so a session whose process left no registry entry reads `unknown` even when its run finished.
4. An unreadable events file, then no signal past the prompt offset: `unknown` with that cause.
5. The signal-derived states above, the interrupt marker turning `busy` into `idle-stalled` as stated.

The idle notice changes no state except through that contradiction.
A state change is recorded only when the state or its cause changes, stamped (its `since`) with the time of what caused it:

- The causing signal's `lyx_at`: a turn start or a waiting turn end (`busy`), `StopFailure` (`api-error`), an ask signal (`asking`), a session end (`dead`, or `unknown` for `clear` and `resume`), and the idle notice for the contradiction (`unknown`).
- The turn end's `lyx_at`: `idle-done` judged at the reading that folded the turn end, `no-output`, interactive `asking` from a missing output file, and a transcript-marked `api-error`.
- The transcript entry's own timestamp: `interrupt`.
- The time of the reading, for a state that rests on a call-time fact: `dead` `process-gone`, `unknown` `liveness-unproven`, `unknown` from no signal or an unreadable events file, and, in the wait loop only, `idle-done` reached at a later poll because output files appeared after a `no-output` or interactive-`asking` turn end.
  A files-only reader never produces that last stamp: it folds every turn end with its one stat, so for it an `idle-done` always carries the turn end's `lyx_at`.
- Empty, when the causing line has no time (D1).

The history is ordered by signal order, and its `since` values are not guaranteed to increase: an `interrupt` entry carries the transcript's own clock and the others the hook's, at second resolution.
The entries resting only on signals (turn starts other than the newest, waiting turn ends whose tasks the Stop payload reports, a `StopFailure` `api-error`, asks, session ends, the contradiction) are the same for every reader.
The entries resting on a reading-time check are per reader, because each reader makes the check itself, and record-on-change makes the entry after one per reader as well, since whether it is a change depends on the entry before it:

- An output-file check (`idle-done`, `no-output`, interactive `asking`): the wait loop judges with the files as they were at each poll, a files-only reader with the files as they are at its call, so a files-only reader that finds the outputs present reads an earlier no-output turn end as `idle-done`.
- A transcript marker (`interrupt`, a transcript-marked `api-error`): each reader marks only the newest turn start or turn end of its own fold, so a reader that folds after the transcript has moved on never records an entry an earlier reader recorded.
  The newest turn start's entry is therefore per reader: `idle-stalled` `interrupt` for a reader that finds the marker, `busy` `turn` for one that folds after a later prompt.
- A transcript-fallback turn end (tasks the Stop payload does not report): its outstanding tasks are read from the transcript at parse time, so a reader that parses after a task's completion notification reached the transcript reads that turn end without the task, as `idle-done`, `no-output` or interactive `asking` where an earlier reader recorded `busy` `background`.
- A call-time fact: no writer persists these; a files-only reader (`lyx shuttle state`, batten, the orch watcher) restamps such a state with the time of each call, while the wait loop logs it once at the change.
  The newest history entry before it that is not itself a call-time state bounds how long it has held, carrying the `since` this list gives it.
Shadow mode compares each site's own reading with its own guess at that moment, which needs no persisted history.
Rationale: every state rests on a hook, the process or a marked transcript entry, never on elapsed time, mtime or pane text; an autonomous agent's question goes to its parent through the existing held notice, so without an explicit ask signal a missing-output turn end is `idle-stalled`, and only an interactive run has an operator waiting at the pane.
Telling "asked" from "stalled" by the message text would be a guess, which the board rules out.

### D4 — Liveness from the provider process

Decision: liveness is the Claude process of the session: the engine finds it through Claude's session registry entry for the session id (pid and `procStart`, the lookup `resume.go` already does) and reads it alive when the pid is alive with a matching start time.
No registry entry, an unreadable start time or a reused pid is `unknown` (cause `liveness-unproven`), not `dead`; a dead pid is `dead`.
The session id looked up is D3's: the newest hook line's, else `run.json`'s.
Claude Code appears to remove its registry entry when a session exits normally (every entry in one machine's `~/.claude/sessions` named a live pid), so a normally exited process mostly reads `unknown` `liveness-unproven`, and `dead` `process-gone` needs a stale entry a crash or kill leaves; this is unverified and listed under Open risks.
D5 logs the wait loop's `died` against such an `unknown` as a disagreement on purpose: how often it happens is the shadow-mode data that shows whether the registry is enough.
The pane is not consulted by the reducer.
Rationale: the board's "liveness is the process"; reed's pane liveness is the wait loop's existing cross-check and stays where it is.

### D5 — Shadow logging at today's guessing sites

Decision: three sites log the state and never branch on it:

- Shuttle's wait loop folds new signals on each poll it already does and logs every state change at Info (`shuttle: session state`), with state, cause, since, and its own current classification (running, waiting, held, done, died).
  When the two disagree under a fixed mapping (waiting↔`busy`, held↔`idle-stalled`/`asking`, done↔`idle-done`, died↔`dead`, running↔`busy`), it logs `shuttle: session state disagrees` at Warn, once per disagreement.
  Three pairings the wait loop produces by design are expected, not disagreements, and log nothing beyond the state change: a gate wait (waiting, with every output present) with `idle-done`; a held turn end after transcript-reported shells expired with `busy` cause `background`; and the wait loop's `done` with `busy` cause `background`, which covers D8's Done-at-once and an ungated run's Done with a fork or shell still outstanding.
  The wait loop's classification wins wherever the two differ: the state is shadow and never overrides it.
- The orch watcher, on each idle probe it already makes, reads the orch session's state through a new `Runner` method, from the orch's run record and events file, and logs at Warn when the probe's idle answer disagrees with it (idle vs `busy`, busy vs an idle state).
- Batten's activity reading (`battencli`'s `readAgentActivity`) reads each live run's state beside its activity reading and logs it at Info when it would send a quiet or API-error notice, so each notice can be checked against the state.

Rationale: shadow mode's purpose is reading disagreements against real runs; logging where each guess is made pairs every guess with the state at that moment.
Bound: these sites only log; a failure to read the state is logged and changes no decision, notice or return.

### D6 — `lyx shuttle state`

Decision: a read-only verb under the existing `shuttle` cobra subtree, `lyx shuttle state`, printing JSON for every run under the current worktree's run-directory root whose record reads `running`, whether or not its waiting process is still alive, so a run whose Wait died shows as `dead` or `unknown`: strand name, state, cause (an `api-error` cause carries the error text), since, and the change history; `--strand <name>` narrows to one.
For a state from a call-time fact, `since` is the time of the call (D3), and the newest history entry before it that is not a call-time state shows when the session's state last changed otherwise, with the `since` D3 gives that entry.
It is wired through `cliwire` like its siblings, reads files only, writes nothing, and is exercised by the sandbox suite.
Rationale: the operator and the hub need to see a run's state without grepping logs while shadow mode is being judged; it is also the reducer's public test surface.
Bound: it reads only the current worktree's run directories and Claude's session registry, prints only what those files already hold to a reader of the worktree, and changes no file, run or session.

### D7 — A held turn end shows as a wait

Decision: while the wait loop holds a turn end, it shows a third wait label, `held`, through the same `wait.yaml` marker and `ReedOps.SetWaitMark` pane mark as the gate and background-shell waits, started at the held turn end and cleared at the next event, at any finalize and at Wait's return.
`lyx loom status`'s waiting line shows it like the others.
Batten's `waitLive` counts only gate and background-shell waits: a `held` mark does not suppress batten's quiet notice.
Rationale: run-recovery made every wait visible, and a held session without a mark looks like a hang; a held session is exactly what the quiet notice exists for, so it must not hide it.
Bound: the mark stays display and status only (`PATTERN-wait-mark-display-only`); the only reader that branches is batten's `waitLive`, which this decision narrows, never widens.

### D8 — A turn end waiting on live background work is a wait (#429)

Decision: in the wait loop, a turn end whose outstanding work is reported by the Stop payload (`SignalPayload`) keeps the run waiting for as long as each new turn end lists it running, bounded only by the run's own Timeout and the liveness check; it is never counted as a turn end, never held and never notified to the parent.
`background_shell_wait_min` applies only to shells reported by the transcript fallback (`SignalTranscript`), where today's expiry rules hold unchanged.
A run without a gate already finishes Done at any turn end where every output file exists, whatever is outstanding and without `ExpiredShells`, and this stays unchanged; the rule below concerns a gated run (webster's Master), whose waiting turn end is not an arrival today even with every output present.
At a gated run's turn end where every output file exists and the only outstanding work is payload-reported shells not matching `Spec.AwaitedShellPrefixes`, the run finishes Done at once instead of waiting out `background_shell_wait_min`; those shells are logged and returned as today's `Result.ExpiredShells`, so webster's friction note still records them.
In a gated run, outstanding forks and awaited-prefix shells keep the run waiting as today.
Rationale: since `shuttle-turns` the payload is authoritative, so a payload-reported running shell is live work, and a long measurement or simulation must never read as a stall; Claude Code wakes the session itself when the shell ends.
The transcript fallback is inference, so it keeps its bound.
With outputs present the agent's step is done, and a leftover shell is not a reason to delay the gate.
Bound: a payload-reported shell that never ends now holds the run until its Timeout (for Master, `master_timeout_min`) instead of `background_shell_wait_min`, and the run then ends `OutcomeTimeout` exactly as a timed-out run does today; lyx reaps no shell itself.
While only non-awaited shells are outstanding, the `background shells` wait mark shows it and batten's quiet notice stays suppressed (D7's `waitLive`); with a fork outstanding beside it, no mark shows, as today, and the quiet notice is not suppressed.
The one added visibility is a log line, `shuttle: background shell past wait min` at Info, once per shell when a payload-reported shell outlives `background_shell_wait_min`, which changes no decision.
Beyond that a hung shell is silent to the parent until Timeout, by choice: a step that never returns is the out-of-scope `verify-watchdog`'s to bound.

### D9 — A stuck outcome is never reported as a question

Decision: no code change beyond a regression pin.
`shuttle-turns` removed the `shedadapters` asking branch that produced `webster master is asking a question`, and a Master that writes `outcome.yaml` with `outcome: stuck` plus `summary.md` now reaches shuttle's Done and webster's stuck result through `mapMasterDone`, with the verify gate passing on a non-done outcome.
The pin: such a Master run, through the webster shed adapter, yields a Stuck step whose reason carries the outcome's `stuck_reason`, and nothing in the result, the halt or the notice text calls it a question; under D3 the session reads `idle-done`.
Rationale: the reported run predates `shuttle-turns`; the bug's path is gone, so the task pins it rather than rebuilding it.

### D10 — One declared table of lyx's denies

Decision: `claudeengine/settings.go` declares lyx's standing denies in one table, each row naming its tool matcher, its match, its steer reason, its system-prompt notice (or none, as for the webster fork guard today) and the config and run inputs that install it; `buildSettings` and `buildDenyNotice` both iterate the table.
The existing denies (Agent, Agent non-fork, AskUserQuestion, webster fork guard) move into it unchanged in behavior.
Rationale: the board's "deliberate permission setup"; the hooks and the notice already must not drift, and a table makes adding a deny one row.
lyx's agents run in bypass mode, so lyx needs no allow list; denies set in the operator's own settings stay the operator's, and `PATTERN-no-denied-recovery` keeps lyx's texts from prescribing them.

### D11 — Deny `python` in Bash

Decision: a new table row denies a Bash call whose command runs `python`, `python3` or `python3.<minor>` in command position: at the start, or after a newline, `;`, `&`, `|`, `(`, `$(`, a backtick, `env`, `exec`, `xargs`, `sudo` or `time`, optionally path-prefixed (`/usr/bin/python3`).
The hook, like the webster fork guard, greps the raw PreToolUse payload JSON, not a parsed `tool_input.command`: "the start" is the start of the `command` value (`"command":"`), and "a newline" is the JSON escape `\n`.
A `python` word as an argument (`grep python file`, a path containing it) is not matched.
The deny is a guardrail against an agent's habitual Python use, not a barrier against an agent set on running it.
It lets through: Python behind a shell wrapper (`bash -c`, `sh -c`, `eval`), after a prefix not in the list (`env FOO=1`, `nohup`, `command`, `timeout`, a shell keyword such as `then`, `do`, `{`, `!`), through a launcher (`py`, `uv run`, `uvx`, `pipx run`), and any other scripting runtime.
It falsely denies text that puts `python` after a listed separator inside a quoted argument, a heredoc body, or another payload field such as the call's `description`.
Its steer: lyx agents never run Python; edit files with Edit or Write, search and read with grep, awk or cat.
It is announced in the deny notice and installed for every lyx-spawned session, interactive and autonomous, forks included, under a new `shuttle.yaml` key `claude_deny_python`, default `true`.
`cmd/lyx/norecoverydeny_test.go`'s denied-form list gains `python` and `python3`, and `pattern/PATTERN-no-denied-recovery.md` names this task's table as the owner of the real set.
Rationale: the operator's rule that lyx agents never run Python.
Bound: it can block a legitimate Python run in a Python target repo, which sets `claude_deny_python: false`; verify commands run through `verifytree` in Go, not the agent's Bash tool, so the deny never touches them.
The switch is read by `claudeengine`'s `Prepare`, from the `Config` its spawning shuttle runner loaded from the worktree's `shuttle.yaml`, when it writes a session's `settings.json` at spawn; a resume relaunches with that same file.
So an agent that edits `shuttle.yaml` mid-run changes no running session's deny, but every session spawned afterwards in that worktree reads the edited value: an agent that can write `shuttle.yaml` can turn the deny off for later sessions, which is within the guardrail-not-barrier bound above.

### Added after Discussion (parent, 2026-10-08): Batten activity ignores lines ParseEvents skips

Decision: Batten's activity reading must not count the new hook and stamp lines: ReadAgentActivity takes the events file's activity from the newest line ParseEvents parses (Stop or ask), not from the file's mtime. D1's Bound names every reader of the events file's mtime or size, and an acceptance row pins that a Notification idle_prompt append does not move LastActivity.

Rationale: Shadow mode must change no behavior; an idle_prompt append restarting batten's quiet clock for a stalled session is a behavior change.

### Added after Discussion (parent, 2026-10-08): Incremental reader holds a trailing stamp

Decision: An incremental reader does not advance its offset past a stamp line with no payload line after it; the stamp is re-read with its payload on the next poll. An acceptance row splits a batch between a stamp and its payload and expects the same since as a whole-file read.

Rationale: Keeps since the same for every reader, as D1 claims, without new shared state.

## Constraints

- `PATTERN-shuttle-provider-seam`: hook names, payload fields, the registry and the transcript markers live only in `claudeengine`; the states, the reducer and the signal types are provider-neutral in `shuttleengine`.
- `PATTERN-completion-signal`: the output-derived negative states (`no-output`, interactive `asking`) consult the output files first, and the first consumer that acts on `dead` applies the output check before acting (D3); the reducer adds no done-when writer or reader.
- `PATTERN-wait-mark-display-only`: the `held` pane mark goes through `SetWaitMark` from `shuttleengine` only; no Go decision reads pane options.
- `PATTERN-told-geometry`, `PATTERN-cwd-resolution`, `PATTERN-lyxdirs-single-declarer`: the reducer and readers are handed run directories; only `shuttlecli` resolves the worktree, through `cliwire` (`PATTERN-cliwire-sole-wiring`, `PATTERN-cli-cobra`).
- `PATTERN-orch-pane-single-writer`: the orch watcher's shadow log types nothing.
- `PATTERN-no-denied-recovery`: the denied list grows by `python`/`python3`; no stencil or spec names them as a step.
- `PATTERN-sandbox-coverage`: the sandbox suite exercises `lyx shuttle state`.
- `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-test-economy`: the reducer and parse tests are untagged with an injected clock; the hook-command round trip through a real shell is `integration` tier; hook-payload capture is `llm` tier through `llmkit`; corpus cases extend the existing table test.
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
- D1's `Notification` `idle_prompt` append moves the events file's mtime after an idle stretch, delaying batten's quiet notice by up to Claude Code's idle threshold; whether Claude Code repeats the notice within one idle stretch is unverified, and if it does, each repeat delays the quiet notice again.
- D1's new hooks add appenders to the events file; two hooks of one session appending at once can land in today's gap between a payload and its newline, and a Stop line lost that way holds the run to its Timeout, as it does today.
- Whether Claude Code removes its session-registry entry at a normal exit, and whether the entry follows the new session id after `/clear` or `/resume`, is unverified; either way liveness degrades to `unknown` `liveness-unproven`, never to a false `dead`.
- The python matcher is a regex over the raw JSON payload; its let-through and false-deny classes (D11) are accepted, and an agent that routes around it, or edits `shuttle.yaml` for later sessions, is not stopped.

## Acceptance criteria

- A session's events file gains one line per `UserPromptSubmit`, `StopFailure`, `Notification` and `SessionEnd` hook, and every hook's payload line, Stop and the ask record included, is preceded by a stamp line naming its hook.
  Run through a real shell (`integration` tier), the Stop and ask record commands append a 20000-byte payload, past the shell's 4096-byte `printf` chunk, byte-identical to the input, and `ParseEvents` yields the same events from the file as from the payload alone, `Raw` included.
  A payload line with no stamp of its hook, or whose stamp's `lyx_at` is empty or malformed, yields an empty `since` for every reader; a failed stamp still appends the payload; two stamps of different hooks before their payloads pair each payload with its own hook's stamp; a stamp read at one poll and its payload at the next pair as in a whole-file reading.
  An `idle_prompt` append to an idle session's events file moves `ReadAgentActivity`'s `LastActivity` to the append's time, the one activity change D1 accepts.
  No recording hook prints output; the four new hooks exit 0 on a failed append, and Stop and the ask record exit 1 as today.
- Every existing corpus case replays through `ParseEvents` and the wait loop with unchanged readings, and each case also states the session state its events reach.
- The reducer, over replayed signals, reads: a turn start → `busy` `turn`, and a second turn start records no entry; a waiting turn end → `busy` with its tasks; a turn end with outputs → `idle-done`; a turn end in a session with an empty output list → `idle-done`; the orch session's turn end, read from its own run record → interactive `asking`; an autonomous no-output turn end → `idle-stalled` `no-output`; an interactive one → `asking`; an AskUserQuestion or permission notification → `asking`; a `StopFailure` → `idle-stalled` `api-error` with its text; an API-error transcript entry without `StopFailure` → the same; an interrupt marker after a turn start → `idle-stalled` `interrupt`; a dead registry pid → `dead`; a session end with reason `logout`, `prompt_input_exit` or `other` → `dead`; a session end with reason `clear` or `resume` and a live process → `unknown` with that reason, until the next turn start reads `busy`; the liveness lookup after a `/clear` uses the newest hook line's `session_id`, not `run.json`'s; no registry entry, a reused pid or an unreadable start time → `unknown` `liveness-unproven`; an idle notice after a turn start → `unknown` contradiction; no signals → `unknown`; signals only before the prompt offset → `unknown`; a no-output turn end whose output files have since appeared → `idle-done`, never a negative state.
- D3's precedence holds: a process-ending session end with the process gone → `dead` with the session-end reason; a dead pid after a turn end with outputs → `dead` `process-gone`; no registry entry after a turn end with outputs → `unknown` `liveness-unproven`.
- The change history holds one entry per state-or-cause change, stamped as D3 states for each cause, and none for a repeated identical signal; a files-only reading whose outputs exist stamps `idle-done` with the turn end's `lyx_at`, and only the wait loop stamps it with the reading time when the files appear at a later poll with no new signal, re-judging only the newest turn end.
- A transcript marker labels only the newest turn end or turn start of a fold: a files-only reading of an older turn end followed by a newer one never gives the older one the transcript's API-error marker, and a reading after a later prompt records no `interrupt` for the earlier turn.
  A transcript-fallback turn end whose task's completion notification has since reached the transcript reads, in a later files-only reading, without that task.
- An interrupt marker reads `idle-stalled` `interrupt` by transcript order alone: a turn start whose `lyx_at` is later than the marker's timestamp, or missing, changes neither the state nor the cause.
- The `fork-killed-api-error` corpus case reads `idle-stalled` `api-error` at its API-error turn end.
- The wait loop logs each state change and logs a disagreement once when its own classification and the state differ under D5's mapping; D5's three expected pairings (gate wait with `idle-done`, post-expiry hold with `busy`, `done` with `busy` cause `background`, gated or not) log no disagreement; no wait-loop verdict, notice or return changes because of the state.
- The orch watcher and batten log the state beside their judgment, and a state read failure changes none of their decisions.
- `lyx shuttle state` prints each running record's state, cause, since and history as JSON, including a run whose waiting process is gone, `--strand` narrows it, and it writes nothing.
- A held turn end writes `wait.yaml` and sets the pane mark with label `held`, both cleared at the next event and at return; `lyx loom status` shows it; batten's quiet notice still fires for a held run.
- A turn end listing a payload-reported running shell keeps an autonomous run waiting past `background_shell_wait_min` with no held notice, logs `shuttle: background shell past wait min` once for that shell, and the run continues at the agent's next turn end; if no turn end comes, the run ends `OutcomeTimeout` at its own Timeout.
- A transcript-reported shell still expires after `background_shell_wait_min` as today.
- In a gated run, a turn end with every output file present and only payload-reported unawaited shells outstanding finishes Done at once and names those shells in `ExpiredShells`; with an outstanding fork or awaited-prefix shell it keeps waiting.
  An ungated run with every output file present finishes Done as today, whatever is outstanding.
- A webster Master run that writes `outcome: stuck` and `summary.md` yields a Stuck step carrying the stuck reason, and no result, halt note or notice text calls it a question.
- A Bash call running `python3 -c …`, `cd x && python script.py`, a second line starting `python3`, `/usr/bin/python3 - <<EOF` or `xargs python3` is denied with the steer; `grep python file` and `ls python-tools/` are not; `bash -c 'python3 x'` and `uv run x` are not, as accepted misses; the deny is installed for interactive, autonomous and fork-enabled sessions alike; `claude_deny_python: false` installs no python deny; the deny notice announces it exactly when it is installed.
- The deny table drives both `buildSettings` and `buildDenyNotice`; each existing deny's hook command and notice sentence is byte-identical to today's, and the notice differs from today's only by the python sentence.
- `norecoverydeny_test.go` fails on a stencil that offers `python3` as a step.
- An `llm`-tier scenario captures real `UserPromptSubmit`, `Notification` (`idle_prompt`, the only type a bypass-mode session reaches) and `SessionEnd` payloads, and its events reduce to the states the run went through.

## Notes for the plan writer

- `internal/shuttleengine/activity.go`'s `ReadAgentActivity` and `runIsLive` are the model for a files-only reader over a run-directory root; the API-error transcript reading is `claudeengine`'s `ActivityReader`.
- Claude's session registry lookup is in `claudeengine/resume.go` (`registryEntry`, `checkResumable`).
- The hook commands and the existing deny/notice drift guard are in `claudeengine/settings.go` (`buildSettings`, `denyInstalls`, `buildDenyNotice`, the steer-character panic check).
- Claude Code 2.1.294's hook names include `StopFailure` (fields `error`, `error_details`, `last_assistant_message`), `Notification` (`notification_type` such as `idle_prompt`, `permission_prompt`), `UserPromptSubmit` and `SessionEnd`; the Stop payload also carries `session_crons`.
- The background-shell expiry and `ExpiredShells` live in `internal/shuttleengine/wait.go`; webster's friction note on expired shells is in `internal/websterengine/runlevel.go`.
- Batten's `waitLive` comes from `battencli/wire.go`'s `readAgentActivity` and is read in `battenshed/notice.go`.
- The orch watcher's idle probe is `orchengine.Watcher.probeIdle`; the orch session has an events file but no wait loop, so its state is read through a `Runner` method beside `SessionIdle`.
