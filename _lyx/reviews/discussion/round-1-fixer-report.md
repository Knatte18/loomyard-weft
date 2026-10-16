# Round 1 fixer report — discussion

## Changes

- **F1** (BLOCKING), fixed as proposed.
  Premise confirmed: `internal/dotgit/testmain_test.go` is `package dotgit` and calls `tmuxkit.Main`, and `go list -deps ./internal/logger` includes `internal/dotgit`.
  D13 now has the logger's stderr half default to `io.Discard` under `testing.Testing()`, the switch `internal/logger/sink.go:175` uses for the durable half.
  Dropped the tmuxkit sentence and rationale, deleted the `PATTERN-testkit` Constraints bullet, kept the `-v` trade-off, restated the acceptance row as a package test run, and updated the Scope bullet.
  Support-log Interview item 5 no longer cites `tmuxkit.Main`.
- **F2** (MEDIUM), fixed as proposed.
  D11's finish line lists Warn sites and Info-or-above lines in a loop or polled path, each with a verdict, and states that a Warn site in no loop needs no per-site verdict.
  The acceptance row now checks that no listed line logs more than once while its state is unchanged.
- **F3** (MEDIUM), fixed as proposed.
  Premise confirmed: `rowNotifier.send` appends to the notice log before the hold-back check, and `orch-template-role.md:130` holds the next-wake line.
  D16 says the dropped line stays in the hub notice log and the orch learns of the landing at its next wake from the board.
  D17 keeps a next-wake fallback that compares the board's done tasks against the landings handled, and a new acceptance row has the claims test pin it.
- **F4** (LOW), fixed as proposed.
  Premise confirmed: `innerRunProducer.report` returns before `reportStuck` when the reason is unchanged.
  D5 now splits the reason-file write (on any text change) from the log (on a key change), and has each caller pass the key beside the text.
- **F5** (NIT), fixed as proposed.
  Deleted D1's quoted tmux error and landing-timeline sentence, D16's `orch-split` sentence and the trace-file Notes bullet.
  Support-log Interview item 2 now holds the tmux error, the timeline and the trace and `reed.json` pointers, and item 8's `orch-split` entry carries the board-fit reasoning.
- Support log: recorded round 1 under Review rounds.

## Disputed

None.

## Deferred

None.
