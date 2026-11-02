# Card 7 — row-gate-wrap

**Create:**
- `plan:internal/gatefixer#Fixer` -> `type Fixer interface`
- `plan:internal/gatefixer#NewStrandFixer` -> `func NewStrandFixer(deps Deps) Fixer`
- `plan:internal/gatefixer#RowFixer` -> `type RowFixer struct`
- `plan:internal/gatefixer#WrapRowGate` -> `func WrapRowGate(spec shuttleengine.GateSpec, entry string, rf RowFixer) (shuttleengine.GateSpec, error)`
- `plan:internal/gatefixer#rowStore` -> `type rowStore struct`
- `plan:internal/gatefixer#fakeFixer` -> `type fakeFixer struct`
- `plan:internal/gatefixer#TestWrapRowGate` -> `func TestWrapRowGate(t *testing.T)`
- `plan:internal/gatefixer#TestRowStore` -> `func TestRowStore(t *testing.T)`

**Extends:**
- `internal/gatefixer/doc.go`

**Uses:**
- `internal/shuttleengine#GateEntry`
- `internal/shuttleengine#GateResult`
- `internal/loomshed#NewVerifyGate`

**Intent:**
The row gate wrap of D2, with the row fixer's records of D8, in `internal/gatefixer`; card 8 applies it to the three rows and card 9 wires it.

- `Fixer` is the spawn-and-wait seam the wrap and card 13's card gate drive: `SpawnOrAttach(store Store, spec Spec) (*Record, bool, error)`, `Await(rec *Record, timeout, wait time.Duration) (*Result, error)` and `Stop(rec *Record) error`.
  `NewStrandFixer(deps)` returns the production one over card 1's `SpawnOrAttach` and `Await`, its `Stop` calling `deps.Stopper.StopStrand(rec.StrandGUID)`.
- `RowFixer` carries `Fixer`, `VisitDir func() (string, error)` (the row visit's directory, D8), `Spec func(reportPath, replyPath string) (Spec, error)` (the caller's prompt and spawn values), `Timeout time.Duration` (gate.yaml's `fixer_timeout_min`), `Forked bool` (whether the row's run evaluates `Forked`, which the row decides at build) and `SkipForked func(findings string) bool` (nil skips nothing).
- `rowStore` is a `Store` over `<visit dir>/<entry name>/records.json`, a list of `Record` written and read through `internal/state`, which keeps unknown keys; it also counts the set's records and marks a record terminal by guid with a status and dead reason.
  A row fixer's report and reply are `report-<n>.md` and `reply-<n>.md` beside it, `<n>` the record's place in the set.
- `WrapRowGate(spec, entry, rf)` returns a copy of `spec` with every entry's other fields kept (`Name`, `Attempts`, `PassOnCap`, `Reprompt`).
  An `entry` naming no entry of `spec` is refused, naming the entry; it is a fail-loud wiring guard, since card 8 passes the name of the element that carries `fixer`.
- Waiting on a live record, in every closure below, means calling `Fixer.Await` until it answers terminal; when `Await` returns an error, the wait stops the strand through `Fixer.Stop` and marks the record dead `died`, so a record whose engine or events can no longer be read never blocks the visit; when that stop fails too, the record stays live and the closure returns the joined error, an infrastructure fault per shuttle's gate contract.
  Every closure that waited on a record marks it terminal with the fixer's status once its own closure has returned, so no live record outlives the evaluation that waited on it.
- The fixer entry's `Gate`, and its `Forked` when the inner one is set, are wrapped:
  1. With a live record in the visit set, it waits for that fixer, runs the inner closure once, marks the record terminal and returns the result, spawning nothing; a failure carries the reply or dead note (step 5's text).
  2. Otherwise it runs the inner closure; a pass, an error or a `Terminal` result is returned as it is.
  3. On a failure it spawns nothing when `SkipForked` reports true for a `Forked` evaluation's findings, or when the set already holds `Attempts + 1` records; it returns the failure as it stands.
  4. Otherwise it writes the findings to the next report file and calls `Fixer.SpawnOrAttach` with the `rowStore` and the caller's spec.
     A spawn error returns the inner failure with one more line naming the spawn error and that no fixer ran, so the writer is re-prompted as today (D1's no-fixer path) rather than the row ending on an infrastructure fault.
  5. After a spawn it waits for the fixer, runs the inner closure again and marks the record terminal with the fixer's status only now, after the re-run, so a driver restarted during the re-run still finds it live.
     A re-run failure's `Findings` is the new findings, then a `## Cold fixer reply` section holding the fixer's reply, or a note that the fixer ended dead with its reason and that any uncommitted change it left is in the worktree.
  The inner gate run has returned before the spawn, so no gate slot is held while the fixer works.
- The fixer entry sets `MayHold`, so shuttle accepts its `Final`, and `Final`: a closure that waits on a live record of the set, if any, then runs the inner closure once, `Forked` when `rf.Forked` and the inner one is set, else `Gate`; it marks a record it waited on terminal, folds its reply or dead note into a failure, and spawns nothing.
  The wrapped closures never return pending, so `MayHold` changes nothing else.
- The fixer entry sets `Expired`, read-only: true when the set holds a live record whose strand this wrap did not spawn, which a driver restarted onto the visit sees, since every attach of a run is a fresh driver process and a fresh wrap; false for a terminal or absent record and for one this wrap spawned and still waits on.
  The wrap keeps the guids it spawned in memory for that test, so no comparison of clock readings decides it.
- Every other entry's `Gate` and `Forked` are wrapped wait-only: with a live record in the set it waits for the fixer, runs its own closure, marks the record terminal and, on a failure, folds the reply or dead note into the findings; with none it runs its closure at once.
  An evaluation runs entries from the first, so whichever entry runs first does the waiting and every entry is judged over the fixer's finished tree.
- A `VisitDir` or store error is returned as the closure's error, an infrastructure fault per shuttle's gate contract.

Bound: one fixer per failed evaluation at the idle boundary, none for a failure judged over a fixer the same evaluation waited on, none from `Final`, and at most `Attempts + 1` per row visit, which bounds the re-sends `handleGatedBoundary` makes without spending an attempt; each fixer is bounded by `Timeout`.
A row run outruns its deadline by at most one fixer timeout plus three runs of the inner gate, since `Final` spawns nothing.

Tests, tier0, over `fakeFixer` (a scripted `Fixer` that records spawns and stops, writes a reply or ends dead, answers running a set number of times before terminal, can fail its spawn, its `Await` or its `Stop`, and can leave a live record behind) and scripted inner closures in a temp visit dir.
`TestWrapRowGate` rows:
- a passing inner spawns nothing; a `Terminal` inner failure and an inner error are returned unchanged and spawn nothing;
- a failure spawns one fixer with the findings in the report, waits through an `Await` that answers running first, re-runs, passes and marks the record terminal;
- a forked evaluation whose findings `SkipForked` rejects spawns one fixer and re-runs the inner `Forked` closure, and one `SkipForked` accepts spawns none;
- a re-run failure carries the new findings and the reply, and a dead fixer's re-run failure carries the dead note with its reason;
- a spawn error returns the inner failure with the spawn line, leaving no record;
- the set at `Attempts + 1` spawns none;
- the fixer entry's `Gate` with a live record waits, re-runs once, marks it terminal and spawns nothing;
- an `Await` error stops the strand and marks the record dead `died`, and a failing stop beside it leaves the record live and returns the joined error;
- `Final` with no live record runs the inner closure once and spawns none; `Final` with a live record waits, runs the inner `Forked` closure when `Forked` is set and `Gate` otherwise, marks the record terminal and folds the reply into a failure;
- a wait-only entry with a live record waits before its closure, marks the record terminal and folds the reply, or a dead fixer's note, into its failure; with none it runs at once;
- `Expired` is true for a live record a fresh wrap did not spawn, false for a terminal record and for one this wrap spawned;
- the wrapped entries keep `Name`, `Attempts`, `PassOnCap` and `Reprompt`;
- an unknown entry name is refused; a `VisitDir` error is returned as the closure's error.

`TestRowStore`: records round-trip through `records.json`, an unknown key in the file survives a rewrite, `Live` returns only a record with no terminal status, and marking a guid terminal leaves the others unchanged.

**ImpactSummary:** new members of `internal/gatefixer` only, with no caller until card 8 and card 9; the package doc gains the wrap.
