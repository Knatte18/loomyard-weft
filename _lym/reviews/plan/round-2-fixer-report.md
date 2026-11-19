---
dispositions:
  - id: F16
    disposition: fixed-differently
    reason: "The proposed glyph kept an error return on `hardenLiveRun`, which takes only an exit code and two byte slices and so has no spawn or wait that could fail; the glyph returns `hardenshed.LiveRun` alone and the `Run` seam owns the spawn or wait error. The self-check also showed that the proposed `Refused` rule misreads a call halted after a tier ran (`internal/gatecli/run.go:590`, `run.go:668`, `run.go:762` print an `ok` false envelope too), so `Refused` also requires that stderr carry no started line."
---

## Changes

- F16, fixed differently.
  - `_lym/plan/14-loom-harden-wiring.md`: the glyph is `func hardenLiveRun(exitCode int, stdout, stderr []byte) hardenshed.LiveRun`.
    The envelope paragraph is replaced: passed is exit 0; the output is stderr followed by stdout, kept whole; `Refused` is the error of stdout's last non-empty line when it is a JSON object with `ok` false and a non-empty `error` and `gateslot.ParseStartedLines` finds no started line on stderr; every other nonzero exit is a failed run; the `Run` seam returns an error for a spawn or wait failure alone, never for a nonzero exit.
    The no-envelope error sentence is gone.
  - Not made as proposed:
    - The `error` return on `hardenLiveRun`. Its inputs hold no spawn or wait, so the error could never be non-nil.
    - The bare "which only a refusal prints, so build output on stdout never matches" claim. A guard build failure (`run.go:590`) and a slot acquire failure (`run.go:762`) also print an `ok` false envelope. A tier's test output also reaches stdout (`internal/gatecli/tierrun.go:82`).
    - Keying `Refused` on the envelope alone. A call halted after a tier ran would then skip the red report.
      Card 1 (`01-gate-copies.md:47`) has every reported tier run print a started line, so its absence marks a call halted before any tier, which is card 6's definition of `Refused`.
  - `TestHardenLiveArgs` rows: exit 0 with a summary on stderr and an empty stdout reads passed; exit 1 with a failure report on stderr reads failed with no `Refused`; exit 1 whose last stdout line is a refusal envelope and whose stderr holds no started line reads `Refused`; exit 1 whose last stdout line is an `ok` false envelope after a started line reads failed with no `Refused`; exit 1 whose stdout holds a JSON line without `ok` false and an `error` reads failed.
  - `_lym/plan/06-live-gate.md`: a `Run` error is "a spawn or wait failure" alone; the `TestLiveGate` rows are unchanged.

No target or verify line changed.

## Self-check

- fix-consistency: no findings.
- fix-affected-set: no findings; nothing unchecked.
- fix-proposal, MEDIUM, `Refused` set for a call halted after a tier ran: fixed with the no-started-line condition and a test row.
- fix-proposal, LOW, test output on stdout makes "build output never matches" incomplete: fixed; the card names build and test output and why neither reads as `Refused`.
- fix-claims, MEDIUM, "which only a refusal prints" is false for guard build and slot acquire failures: merged with the fix-proposal MEDIUM finding and fixed; the card names those halts as `Refused` when they come before any tier, which matches card 6's "refused before running any tier".

## Disputed

None.

## Deferred

None.

## Rulings

None.
