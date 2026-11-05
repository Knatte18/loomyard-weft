# Card 19 — drift-amend-defer

**Create:**
- `plan:internal/driftengine#AmendRequest` -> `type AmendRequest struct`
- `plan:internal/driftengine#Amend` -> `func Amend(d Deps, st VerbStatus, req AmendRequest) error`
- `plan:internal/driftengine#TestAmend` -> `func TestAmend(t *testing.T)`
- `plan:internal/driftcli#amendCmd` -> `func amendCmd() *cobra.Command`
- `plan:internal/driftcli#openSlugWorktree` -> `func openSlugWorktree(cmd *cobra.Command, slug string) (*worktree, error)`
- `plan:internal/driftcli#checkParentCaller` -> `func checkParentCaller(verb, strand string, parent func() (hubgeom.Parent, error)) error`
- `plan:internal/driftcli#TestCheckParentCaller` -> `func TestCheckParentCaller(t *testing.T)`
- `plan:internal/driftcli_test#TestAmend_RefusesWhatItCannotRead` -> `func TestAmend_RefusesWhatItCannotRead(t *testing.T)`
- `plan:internal/driftengine#recheckRow` -> `const recheckRow = "Drift-Recheck"`

**Edit:**
- `internal/driftengine#Defer` -> `func Defer(d Deps, st VerbStatus) error`
- `internal/driftengine#TestDeferWaive`
- `internal/driftengine#TestDriftAwait`
- `internal/driftengine#TestDriftBaseline`
- `internal/driftengine/doc.go`
- `internal/driftcli#deferCmd`
- `internal/driftcli#Command`
- `plan:internal/driftcli#worktree.checkDeps`
- `internal/driftcli/doc.go`

**Uses:**
- `plan:internal/driftengine#ParseExpectation`
- `plan:internal/driftengine#Expectation.Cover`
- `plan:internal/driftengine#WriteRecords`
- `plan:internal/driftengine#RestoreRecords`
- `plan:internal/driftengine#CheckSeal`
- `plan:internal/driftengine#CheckBaseline`
- `plan:internal/driftengine#PastRow`
- `plan:internal/driftengine#expectRow`
- `plan:internal/driftengine#darnWriteRow`
- `plan:internal/driftengine#websterRow`
- `plan:internal/driftengine#DeferHaltPrefix`
- `internal/orchcli#NearestLiveParent`
- `internal/fabricengine#WorktreePath`
- `plan:internal/fabricengine#CommitAnchoredPathsWithTrailers`
- `plan:internal/driftengine#commitStore`
- `plan:internal/driftengine#commitStore.deps`
- `internal/driftcli_test#lymRun`

**Intent:** D11's parent amendment and D12's wider `lym drift defer`.

`checkParentCaller` is the caller check both verbs run, as `lym loom decision add` runs its own: the caller's strand name, `agentname.StrandNameEnv`, empty is the operator's own shell and passes; a name equal to the run's parent orch, resolved by `orchcli.NearestLiveParent` over the target worktree, passes; any other is refused, `drift <verb> refused: the caller "<name>" is not the run's parent; way forward: ask <parent> to run it; the operator's own unnamed shell may always run it`, and an unresolvable parent is refused naming the error and the operator's shell.
Bound: the check rests on the strand name and on the shell guard that refuses setting lym's strand variables (D11).

`Amend` and `Defer` name the rows they admit, the writer rows, Drift-Verify, Drift-Recheck and Drift-Expect, through driftengine's own row constants, `expectRow`, `verifyRow` and this card's `recheckRow` and card 16's `darnWriteRow` and `websterRow`, never as repeated literals; driftengine keeps its own row constants, as `verifyRow` and `captureRowBaseline` stand beside `loomshed`'s names today, and imports no `loomshed`.
`Amend` is admitted only at a blocked writer (`Darn-Write` or `Webster`), Drift-Verify or Drift-Recheck, the rows D11 names; a running one of those rows is refused, naming the way forward, waiting for the row's halt that `lym loom status` shows, then amending; any other row, a blocked Parent-Align or Code-Review row among them, is refused naming the rows it admits.
`AmendRequest` holds `By`, `parent` or `operator`, and exactly one of `Entries` (YAML in `expected.yaml`'s shape), `Restore` or `Discard`; a request holding none or several is refused naming the three flags, before anything is read:
- entries are parsed by `ParseExpectation` under the recorded mode against the baseline listing, so an entry D5 or D2 refuses is refused, the parser's message wrapped as `drift amend refused: <the parser's message>; way forward: correct the entry in the --entries file and run lym drift amend <slug> again` (`PATTERN-refusal-way-forward`); the amendment is recorded and the stored rulings of every cell they cover are dropped from `judged.json`, under `command` the case rulings of the cases they name, so those are judged again against the amended `why`;
- `Restore` writes back whatever failed its check: the seal commit's copy over `expected.yaml` after an edited file, and `RestoreRecords` after a records mismatch, and is refused naming the halt's own ways forward when neither failed;
- `Discard` records that the expectation is set aside: the seal check is skipped and containment reads an empty expectation and every amendment.
Every amendment is recorded in `scope.json` with who, when and its entries, restore or discard, in one `WriteRecords` commit, and never edits the seal commit (D11).
Bound: an amendment only adds coverage, restores the sealed file and records, or sets the whole expectation aside; it cannot remove one entry or change the seal; a hand-written amendment is no amendment, since the records check stops on it.

`Defer` takes `Deps` now, since it reads the records and the baseline: it keeps admitting a blocked Drift-Await and a Drift-Baseline capture halt, and adds a blocked Drift-Expect, writer (`Darn-Write` or `Webster`), Drift-Verify or Drift-Recheck, the rows D12 names, whose error, past an optional leading `DecisionHaltPrefix`, opens with `DeferHaltPrefix`, or, on a run whose scope record holds a scope other than `off` and no deferral, whose baseline check `CheckBaseline` fails when it runs, so a scope-`off` run, which has no baseline directory, never admits a deferral that way; any other blocked row, a Parent-Align or Code-Review row among them, is refused naming the halts it settles; it still refuses while a capture holds the lock, which a `lym drift check` or `drift` gate entry's current capture now also takes (card 9), so the refusal names two ways forward: `lym loom resume`, whose row waits on a running main capture, or, when a check holds the lock, waiting for it to finish, then running `lym drift defer` again; it records `awaited_at` and `deferred` through `WriteRecords` (D12).
`deferCmd` takes an optional `<slug>`, resolved through `openSlugWorktree` as `amend` resolves it, so the parent need not stand in the run's worktree, and the cwd's worktree without one; it runs the caller check first and passes the worktree's `checkDeps`; `checkDeps` gains `CommitWithTrailers` over `CommitAnchoredPathsWithTrailers`.
Bound for the slug: it changes only which worktree the verb acts on, and the caller check and `Defer`'s admission run against that worktree's run.
`amendCmd` is `amend <slug>` with `--entries <file>`, `--restore` or `--discard`, operator audience: it resolves the slug's worktree through `openSlugWorktree` (`fabricengine.WorktreePath` then `lymcwd.Resolve`, refusing an unknown slug naming it and `lym board list` as the way forward), runs the caller check, reads the `--entries` file, refusing one it cannot read naming the path, then `Amend`.
Bound for `defer`: after the writer it skips the regression check for the rest of the run; only the parent or the operator runs it, and the verdict states it.

`TestAmend` (tier0, on `commitStore`, `statusAt`, `t.Parallel`) covers: an entry that covers a formerly unexpected cell makes it expected and drops its stored ruling; under `command` an entry naming a case drops that case's stored rulings, and a `keys` entry is refused; an entry D5 refuses is refused, the refusal naming the way forward of correcting the `--entries` file and running `lym drift amend <slug>` again, and nothing is written; a request with none or two of entries, restore and discard is refused naming the three; `--restore` makes a seal check failed by an edited file pass and restores hand-edited records, and with nothing failed it is refused; `--discard` clears a `Warp-SHA` mismatch and an unsealed file, the run then checked against its amendments alone; the seal check passes after an entries amendment; each amendment is one commit carrying `Drift-Records`; a running Drift-Verify, a blocked Parent-Align and a row before Drift-Expect are refused naming the way forward.
`TestDeferWaive`'s rows build their `Deps` through `commitStore.deps`, and it gains rows: admitted at a blocked Drift-Expect whose head moved, at a blocked Drift-Verify whose capture never counted and whose current capture failed, at a blocked Drift-Recheck with a missing baseline directory, each with its error opening `DecisionHaltPrefix` then `DeferHaltPrefix`; refused at a blocked Drift-Verify halted on an unexpected cell, at a blocked Parent-Align, and at a blocked Darn-Write on a scope-`off` run; refused while the capture lock is held at an admitted halt, the refusal naming `lym loom resume` and waiting for the running check to finish, then running `lym drift defer` again.
`TestDriftAwait` and `TestDriftBaseline`, whose rows call `Defer` to settle a halt before resuming, pass it the `Deps` their fixtures already build.
`TestCheckParentCaller` (tier0) covers the operator's empty name, the parent's name, another session refused naming the parent, and an unresolvable parent.
`TestAmend_RefusesWhatItCannotRead` (tier1, external package, the package's hub fixture and `lymRun`) covers `lym drift amend <unknown slug> --discard` refused naming the slug, and `--entries` naming a missing file refused naming the path, each from the operator's unnamed shell.

**ImpactSummary:** Re-signs `Defer`, whose one caller is the `defer` verb on this card; adds the `amend` verb and an engine function; no other package calls either.
