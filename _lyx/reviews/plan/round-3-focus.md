---
round: 3
focus:
  - "Card 14 (`_lyx/plan/14-landing-watchdog-halt.md`) changes `internal/landingshed#verifyGate.verdict`, which Merge-In (`mergein.go`), Finalize (`finalize.go`, via `fz.gate.check`) and Publish all reach. Settle whether every landing producer whose halt reason now names the watchdog window and `busy_quit_sec` has a `contracts/specs/refusal-spec.md` row, named in the card's Intent, that carries the way forward, and whether the chosen row's label and trigger match the producers it serves."
  - "Card 7 (`_lyx/plan/07-pattern-tolerant-records.md`) lands a PATTERN entry saying a rewrite of a Go-written JSON record goes through `internal/state` or a type that keeps its own keys. Settle whether any production read-then-rewrite of a Go-written JSON record, in particular `internal/loomshed` (`readRoundRecord`, `writeRoundRecord`, `finishRound` in `prrework.go`), still drops unknown keys while no card touches it, and whether any card's edit to such a writer states how unknown keys survive."
  - "Card 11 (`_lyx/plan/11-gate-test-watchdog.md`) prints the watchdog line to the verb's stderr, while `internal/gatecli/cli.go`'s RunE hands `runTest` only `cmd.OutOrStdout()`. Settle whether the card names a concrete route by which the watchdog gets a stderr writer, whether that route needs a re-sign arrow on `runTest` or a field on `testRequest`, and whether the card's targets and integration row agree with it."
---

Each entry names a site the plan's latest edits touched, so the reviewer reads the current text of those cards rather than a prior review.
