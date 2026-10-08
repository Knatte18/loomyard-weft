# Fixer report — discussion round 1

Every finding was validated against the code (`waitmark.go`'s `shellWaitActive`, `rundir.go`'s `RunState`, `activity.go`'s `runIsLive`, `settings.go`'s `buildSettings`/`buildDenyNotice`, `battenshed/notice.go`'s `waitLive`, `websterengine/template.yaml`'s `master_timeout_min`), and each premise held.
All twelve are fixed.
`lyx loom validate-discussion` exits 0 after the edits.

## Fixed

- **F1** — D5 now names three expected pairings that log no disagreement (gate wait with `idle-done`, post-expiry hold with `busy` `background`, D8's Done-at-once with `busy` `background`), and states that the wait loop's classification wins wherever the two differ.
  The wait-loop acceptance criterion names the three as non-disagreements.
  D3 is left unchanged so it needs no awaited-prefix input, which the run record does not hold.
- **F2** — D3 replaces "three facts" with a named list of facts and their sources: the run record (`interactive`, `outputFiles`) for mode and output list, with the orch watcher handing interactive with no output contract; a stat at reading time for output presence; `claudeengine` for liveness and both transcript markers.
  It states that the fold is pure and the reading is the caller's.
  The stamping rule now covers every cause: the turn end's `lyx_at` for `no-output` and transcript `api-error`, the transcript entry's timestamp for `interrupt`.
- **F3** — D3 states that the signal-derived history is the same for every reader, and that call-time entries are per reader with no persisting writer, by decision.
  The support log's rejected `state.yaml` alternative is corrected to match.
- **F4** — D11 states that the deny is a guardrail, not a barrier, names its let-through classes (wrappers, unlisted prefixes and shell keywords, launchers, other runtimes) and its false-deny class (quoted or heredoc text), adds newline as a command separator, and states that the switch is read at spawn.
  The criterion pins a second-line `python3` as denied and `bash -c 'python3 x'` and `uv run x` as accepted misses.
  The python Open risks line points at D11 rather than restating the classes.
- **F5** — D8's Bound states the `OutcomeTimeout` result with no shell reaped, batten's quiet notice suppressed while only shells are outstanding, the fork-plus-shell case (no mark, notice not suppressed), and a new Info log line once per shell past `background_shell_wait_min`, which changes no decision.
  It records that a hung shell is otherwise silent until Timeout by choice, with `verify-watchdog` named as out of scope.
  The criterion now covers the log line and the Timeout outcome.
- **F6** — D1 gains a Bound: hook commands print nothing and exit 0 on a failed append; `lyx_at` is advisory, read only by the reducer, and a malformed value reads as first-seen time.
- **F7** — D6 now lists every record that reads `running`, including runs whose waiting process is gone, so `dead` is reachable; it names the `api-error` text in the cause, and gains a Bound on what it reads and exposes.
  The criterion is updated to match.
- **F8** — Goal part 3 is qualified to payload-reported work, and notes that transcript-only shells keep today's expiry and held stop.
- **F9** — The deny-table criterion now reads: each existing deny's hook command and notice sentence is byte-identical, and the notice differs only by the python sentence.
- **F10** — Added rows to the existing criteria: first-seen time for a missing or malformed `lyx_at` and the silent, non-blocking hook (D1); a reused pid and an unreadable start time read `unknown` (D4); output files that have since appeared read `idle-done`, never a negative state (D3); the Timeout outcome (D8); install for interactive, autonomous and fork-enabled sessions (D11).
- **F11** — Removed from the decision record: D9's commit times, D8's PR-Gate attribution and D11's incident names.
  The commit times moved to support-log Interview item 8 and the incident names to item 10.
  The PR-Gate attribution was already in Rejected alternatives.
- **F12** — Q14 now names all three captured payloads and says that `idle_prompt` is the only `Notification` type a bypass-mode session reaches; the llm-tier criterion says the same.

## Disputed

None.

## Deferred

None.
