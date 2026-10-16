# Fixer report — plan, round 2

Every finding's premise was checked against the code and holds; all four are fixed as proposed.
`lyx loom validate-plan` reports `ok` on the edited plan.

## Changes

- **F1** (MEDIUM, scope), fixed as proposed: card 2 lists `internal/reedengine#TestUpdateStrandLocked` under Edit, and its Tests paragraph says the `TopPositionalDisplayRejected` row goes with the content-sized refusal while every other row stays for card 5.
  Confirmed at `internal/reedengine/strand_test.go:262`, whose row wants `content-sized`.
- **F2** (LOW, design), fixed as proposed: card 8's Prosa list gains `windowsize.go`, `watchdog.go`, `attach.go`, `state.go`, `bar.go`, `ownwindow.go`, `windowsize_test.go`, `bar_integration_test.go` and `strandops_integration_test.go` under `internal/reedengine`, and its Intent says each header is rewritten to describe only what its file still holds after cards 1 to 7, naming what each loses.
- **F3** (LOW, consistency), fixed as proposed: card 2 lists `internal/reedengine#TestMultiplexerContract` under Edit and its Tests paragraph says the step (c) comment stops citing `apply_test.go` and `render.Rules`; card 5 keeps its Edit entry and its Tests paragraph says the step (b) comment stops describing the raw `split-window` as `launchStrandLocked`'s path.
  Confirmed at `internal/reedengine/contract_integration_test.go:198` and `:209-210`.
- **F4** (LOW, consistency), fixed as proposed: card 5's Tests paragraph renames `StackedAddsKeepEverySessionPane` to `AddsKeepEverySessionPane` and rewrites its comment; card 8's smoke header edit rewords the scenario as one live pane per strand across successive adds instead of dropping it.

## Disputed

None.

## Deferred

None.
