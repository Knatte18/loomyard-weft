# Red-first holds on parent-only refusals; halts name real files and ways

Three fixes for runs that halted on refusals the writer could not lift, or with halt messages that pointed at the wrong thing (issues #173, #174, #175).

## Red-first holds on a refusal only a commit or the parent lifts (#173)

`lym gate red-first` exits `RedFirstNoTestExit` when the diff adds or changes no test,
and `RedFirstUnjudgedExit` when the hub's scoper is not go.
`redFirstFailure` turned both into ordinary failing results,
so the writer was re-prompted with the same refusal until the entry's attempts ran out.
The gate-dotnet-fixes and red-first-overlay-non-go-files runs spent three attempts in two minutes this way,
while the parent's `lym loom decision add` was still committing.

`loomshed.NewRedFirstGate` now returns a `RedFirstGate` with two closures:

- `Gate` answers either exit **pending**, so the shuttle wait loop holds the run without counting a failure.
  The hold's first arrival sends the writer one line naming the way forward:
  add a failing test, or ask the parent for a `No test seam` record.
  Later arrivals send nothing.
  While HEAD's tree is the one the refusal judged, `Gate` answers pending again without running the check,
  so the 5-second idle re-check never re-runs `lym gate red-first` or fills the verify log directory.
  A new commit runs the check again, and the parent's `No test seam` record lifts the hold through the existing record check.
- `Final` runs at a deadline or liveness finalize.
  It answers the same exits as a failure whose findings name the same way forward, so a run that ends while held still halts with readable findings.

`shedrecipe` builds the `red-first` entry with `MayHold: true` and that `Final`, since `evaluateGate` refuses a pending result from any other must-pass entry.
`verifytree.HeadTree`, the former unexported `headTree`, is exported so the gate can compare trees.
The green exit, a timeout, a runner crash and every other exit behave as before.

## A halt's gate findings file survives a later pass (#174)

Reproduced from the archived raddle-slug-length-check trace.
The Darn-Write halt at 20:48:28 wrote `.lym/shed/<slug>/gate-findings/Darn-Write.md` and named it in the reason.
The tackler resumed the run, and the respawned Darn-Write passed its gate at 20:49:51.
`clearGateFindings` then deleted the file that the halt note still named.
The orphan scratch sweep in `battencli/cleanup.go`, the triage's suspect, plays no part:
it runs on the prime's scratch only.

`clearGateFindings` and its three call sites are removed (`MultiLLMProducer`, `BurlerReviewProducer` and `BurlerFixProducer`).
A later failure still overwrites the file.
The one reader that relied on the clearing, `latestDarnOutcome` in `loomcli`,
now names the kept file in the writer's prior-work note only when the latest Darn-Write entry is a Stuck.
A spawn after a pass is therefore never pointed at old findings.

## Preflight's stencil build-version refusal keeps only its own way forward (#175)

`wireDarn`'s `PreflightStencils` appended "way forward: fix the config or model spec it names, then re-step" to every error from building the writer's table.
A `*stencilstore.NewerBoardCopyError` already ends with its own way forward (restart the driver on a current binary),
so the refusal carried two clauses, the second naming a config fault that does not exist.
The new `preflightTableError` keeps that error's own clause alone and appends the config clause to every other build error.
The error reaches it through `%w` wrapping all the way from `stencilstore`.

## Design decisions

- **The red-first refusals hold instead of halting terminally.**
  The run's added decision "Scope after the corrupted board body" says the gate waits for the parent's decision.
  A terminal result would have halted the run and needed a `lym loom resume` after the record.
  The hold lets the run go on by itself once the record or a test commit lands.
- **The no-test exit holds too, though the writer can lift it by adding a test.**
  The decision names both exits.
  The hold's line names both ways forward,
  and a writer's commit re-runs the check at its next turn end, so the writer loses nothing by the hold.
- **The findings file is kept rather than copied to a per-halt path.**
  A history entry does not record its Stuck reason, so a per-halt path could not be found again for the prior-work note.
  Keeping one file per row, gated on the latest entry being a Stuck, keeps every halt's named path readable
  and needs no new layout.

## Tests

- `TestRedFirstGate_VerdictPerLabelRecordAndExitCode`: the no-test and unjudged exits are pending with a one-line `Send` naming the way forward.
  New rows cover `Final` answering both exits as failures with the old findings.
- `TestRedFirstGate_HoldRunsNothingUntilACommitOrTheRecordLands`: the first refusal holds and sends; a second arrival on the same tree runs nothing and sends nothing;
  a new commit re-runs the check; the `No test seam` record passes.
- `TestResolveGateSpec_RedFirstEntryMayHoldAndCarriesAFinal` pins the entry's `MayHold` and `Final`.
- The multillm, burler-fix and burler-review rows that asserted the findings file was removed on a pass now assert it is kept,
  and so does `shedrecipe`'s `TestBurlerHalfEntries_RunDirectory/ReviewWhoseGatePassedKeepsTheRowsFindingsFile`.
  `TestClearGateFindings` goes with the function.
- `TestLatestDarnOutcome` gains a row: a Done latest entry names no findings file an earlier halt kept.
- `TestPreflightTableError_KeepsANewerStencilRefusalsOwnWayForward`: a wrapped `NewerBoardCopyError` carries exactly one way forward and no config clause;
  any other error gains the config clause.

## Merge danger

**Door:** two-way
**Blast radius:** loom
