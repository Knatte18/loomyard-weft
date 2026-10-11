# Pane injection never merges with an operator draft; batten survives one

Text lyx types into a Claude pane no longer merges with, submits or wipes an operator's draft, and no longer closes a menu the operator has open.
A draft in a parked driver's pane now delays batten's resume instead of failing batten.
A batten run that ends before its teardown while its child loom run is done is flagged and closed by the orch's after-landing routine.

## What changes

- **reed** gains an atomic paste-submit op and an operator-activity query, plus the `operator_quiet_s` key in `reed.yaml`.
  The paste runs as one tmux invocation through a uniquely named buffer deleted with `-d`, so the operator's paste-buffer stack stays untouched and no operator key can land inside the write.
  The activity query reads the attached client's recent activity on the target pane.
  The per-hub watchdog's `/rename` repair holds while the operator is active and uses the paste-submit when it types.
- **shuttle** puts a fresh guard immediately before every write into an agent pane: readiness, no operator activity inside `operator_quiet_s`, and an empty input box.
  This covers `sendVerified`, the start prompt, the skill load, gate sends, `ClearSession`, `CompactSession`, `ReloadPlugins` and `TypeColor`.
  After a paste, an exact match of the sent text (or the paste placeholder for a text long enough to collapse) gets a guarded Enter.
  A box holding the sent text plus other characters, or other text, gets no Enter and no edit, and the send fails wrapping `ErrSessionBusy`.
  A send that may have left text in the box leaves a pending record, so its retry does not type twice when a turn start past the recorded offset shows it landed.
  The prefix match that accepted a 48-character prefix of a longer text is gone.
  `judgeReadiness` holds an unreadable box for an engine that reads its box.
  A start whose skill load or prompt delivery stays held past `send_ready_timeout_s`, or fails on a foreign box, returns a died start naming the cause.
- **claudeengine** declares the paste path and drops the leading Escape and the 300 ms settle from its typing sequences.
  `Runner.Interrupt`, `Runner.Inject`, `InterruptSequence` and `ModelSwitchSequence` are unchanged: they deliberately bypass idleness.
- **orchengine**'s watcher treats a held or foreign-box typed step as a wait: phase and notice are kept, the next tick retries, and the tick is not counted failed.
  A held idle-phase notice records its reason in `State.Stuck`.
- **loom** start and resume refuse a parked driver whose pane holds a draft with the new kind `driver_busy` (refusal-spec row added), leave the park marker, and reuse one resume report path per park (`internal/shedrun`).
- **battenshed** retries `driver_busy` like `ErrChildNotParked`, reports the draft and remedy, and queues one orch notice per stall without quoting the draft.
- **batten status** flags a failed, blocked or paused run whose child loom run is done, naming `lyx batten run <slug>`.
  The orch role stencil's `## After landing` section names that check and the close.
- `PATTERN-pane-text-injection` records the invariant.

## Design decisions

- reed writes and knows nothing of providers; shuttle decides when a write may happen and confirms it landed.
  This keeps the `creel → shuttle → reed → tmux` layering, so creel can later wrap the path.
- A guard failure is a hold, not an error, so callers retry; an activity query that errors logs at Warn and does not hold, while the empty-box check still does.
- The retry of `driver_busy` has no deadline: a draft left for hours stalls the run, visible through the notice and `lyx batten status`.

## Known limits

- A key pressed in the milliseconds between the pre-write capture and the tmux invocation can still precede the paste.
- `client_activity` also moves on mouse and resize events, so scrolling a watched pane delays injection by up to `operator_quiet_s`.
- A slash command left pending and then submitted by the operator is typed again on retry.
- A start held past `send_ready_timeout_s` dies, and its pane goes with any draft typed into it.

## To check by hand

- Run the `llm`-tier smoke test in `internal/shuttlecli/smoke_paste_test.go`: it was compiled with `-tags llm` and not run.
  It decides whether Claude Code submits plain text and `/color` on an Enter right after a bracketed paste end; claudeengine's choice between paste-submit with and without Enter must match its result.
- Run the `tmux`-tier tests in `internal/reedengine` (`paste_integration_test.go`, `activity_integration_test.go`): vetted with `-tags tmux`, not run.
- Review finding `card10-unexported-constant-from-shuttlecli` (consistency, MEDIUM): an unexported constant is reached from `shuttlecli`; check the fix.
- Review finding `card6-foreign-box-error-omits-draft` (consistency, MEDIUM): the foreign-box error in shuttle's send path may omit the draft text it should name; check the wording.
- Raised by hand: `wantSpawnSiteCount` in `internal/shuttlecli/modelpin_test.go` went to 13 for the new real-`claude` spawn site, and the `loomcli` and `battencli` rows in `cmd/lyx/testdata/test-budget.yaml` each grew by one.
