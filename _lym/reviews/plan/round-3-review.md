---
verdict: APPROVED
prior:
  - id: F16
    status: resolved
    answer: "Card 14's hardenLiveRun now takes the exit code and both streams and returns a LiveRun with no error, and card 6 reserves the Run error for a spawn or wait failure. Both match what internal/gatecli emits under --wait, and the fixer's extra started-line condition on Refused is correct."
---

## Review notes

F16 is resolved, and the changed hunks in cards 6 and 14 hold no new BLOCKING defect.
The incremental rule bars a non-BLOCKING finding this round, so the observations below sit under `## Rejected`.

I checked the rewritten reading path against `internal/gatecli` as it stands in the tree:

- With `--wait`, `runTest` runs in the calling process (`internal/gatecli/cli.go:110`, `cappedCall` only takes a call without `--wait`).
  Tier output goes to stdout through `runTier`'s counter (`internal/gatecli/tierrun.go:82`).
  Build output goes to stdout through `runBuild` (`internal/gatecli/build.go:51`).
  The per-tier summary line, the failed-run line and the failure report go to stderr (`internal/gatecli/run.go:683`).
  The exit code is the first failing run's own code (`internal/gatecli/doc.go` "Exit codes").
  An ordinary green or red run therefore reads from the exit code and the two streams, and card 14 needs no envelope for it.
- Only `output.Err` writes an `ok` false JSON object, one line on stdout (`internal/output/output.go:33`), at the refusal sites in `runTest`, `runWithSlots` (`acquireSlot`), `runGuard` and `execution.report`.
  A refusal that `execution.report` answers after an earlier tier reported carries that tier's started line on stderr.
  Card 14's `Refused` rule needs `ok` false, a non-empty `error` and no started line on stderr, so it reads that case as a failed run.
  A tier's own stdout can end in a JSON line of that shape, and the started-line condition keeps it from reading as `Refused`.
- Card 6's `Run` error paragraph now reads "a spawn or wait failure", which is what card 14's seam returns.
  Every branch card 6 prescribes has a named `TestLiveGate` row: a refused `harden-live`, a refused copies call, a `Run` error, green, red, started 0, no started line, red and zero-started copies, the clamp line, the two copies-off conditions and the stray-state cover.
- `TestHardenLiveArgs` has a row for each of: exit 0, a failure report, a refusal with no started line, an `ok` false line after a started line, and a JSON line lacking `ok` false or an `error`.
  Each row's input is parsed as card 14's `hardenLiveRun` text says.

## Rejected

- Card 14's `TestHardenLiveArgs` names no row whose last stdout line is plain, non-JSON build or test output.
  The branch is distinct from the JSON-without-`ok`-false row, but a missing row is not BLOCKING and this round raises only BLOCKING findings.
- Card 1 does not say which stream the started line prints to, while card 14's `Refused` rule reads it from stderr.
  `execution.report` writes its other lines through `e.request.stderr`, and card 14 says plainly that the started lines go to stderr.
  Card 1 is outside this round's changed files.
- Card 14's `Uses` omits `plan:internal/gateslot#ParseStartedLines`, which `hardenLiveRun` now calls.
  Card 14 already omits every other plan-created symbol it calls, and the rubric says plan-time dependency completeness is not provable.
- Card 14's sentence that `Refused` matches "only a call halted before any tier ran" is slightly narrower than the rule.
  A refusal inside the first tier, such as the tier memory-cap refusal, also carries no started line.
  The rule's own conditions decide that case, and the result is a refusal text with its own way forward.
