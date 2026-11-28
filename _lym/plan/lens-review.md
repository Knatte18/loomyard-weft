# Lens review

All five lens forks returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Each report is saved verbatim beside this file as `lens-review.lens-<lens>.md`.
Duplicates across lenses are merged into one line, with every lens that raised the finding named.

- coverage 1 — BLOCKING — fixed differently: the proposal leaves the operand to the writer, so card 7's `joinQueue` sets `TrueMerge` to `!Config.Squash` or a `merge-in` mode, the parents `finalize.go:564` gives today's parent-side merge (an at-once landing never batches). Card 8 gains a row in which a non-squash hub's batch writes two-parent landing commits.
- guard-reach 1 — BLOCKING — fixed differently: the proposal leaves the read-only seam open, so card 8's `memberLandingDeps` replaces the member's `Reconcile` with the new `judgeOnlyReconciler`. It runs `runReconciler`'s reconcile under `fabriclayout.ReconcileScope{JudgeOnly: true}`, which moves no ref and tears nothing down (`internal/fabricengine/originreconcile.go:585`, `originreconcileended.go:120`). A fast-forward that the judge-only reconcile reports leaves the member out. Card 8 gains a row: a member behind origin is left out with its branch ref unmoved.
- forced-edits 1 — BLOCKING — fixed as proposed: cards 7, 8 and 10 now list `internal/testkit/envkit#LandingDeps`, `internal/battencli#intentionallyNil` and every reflective test that marks `internal/landingshed#Deps` under `Edit`. Those tests are `TestLandingDeps_EveryFieldPopulated`, `TestWire_EverySeamFilled` in loomcli and battencli, `TestWire_BeamRecipe`, `TestWire_DarnRecipe`, `TestWire_SamplerRecipe` and `TestFullEnv_FillsEverySeam`. Cards 7 and 8 add the `//lym:seam` markers for `BackstopSeams` and `DetachDeps`, and card 7 names the search that found the tests.
- coverage 2 — MEDIUM — fixed differently: no owner-check code change is needed, because the leader's checks run after its restore and members are never detached. Card 8 now orders the leader's owner checks and `judgeMember` after the gate's restore, so no owner check reads a detached worktree. Card 8 also gains a `TaskHead` branch-ref test.
- coverage 3, row-validity 7 — MEDIUM — fixed as proposed: card 10 adds the tier1 test `internal/loomcli#TestNewShed_RestoresADetachedTaskWorktree` over the `resumeRestore` closure that `newShed` sets, at a row other than Finalize. The landingshed row that claimed to reach the loomcli closure is dropped.
- coverage 4, row-validity 3, forced-edits 2, way-forward 1 — MEDIUM — fixed as proposed: card 4's held-off code push now returns a `deferred`-class `*PushFailedError`, never nil, so the pusher waits it out and does not spin. The error names the marker and `detachWayForward`, and the Bound names `lym loom resume` as the end of a dead leader's hold. Every `readLocalBranch` caller is named, including `TipsOnOrigin` and `PushPairAnchored`. The callers already report a deferred push as owed (`PushPairAnchored`'s doc). The new test `TestPushPairAnchored_HoldsOffTheCodeSideOfADetachedPair` pins the hold-off.
- coverage 5 — MEDIUM — fixed as proposed: card 8 adds `internal/loomcli#TestLandingDeps_TaskHeadReadsTheBranchWhileDetached`.
- coverage 6 — MEDIUM — fixed as proposed: card 8 gains a row in which a gate moves the parent every run with `batch_max` 2, and the leader ends after two rebuilds by landing alone.
- coverage 7 — MEDIUM — fixed as proposed: card 8 gains a row for the `RecordsHead` fallback when `RecordsToHold` fails.
- coverage 8 — MEDIUM — fixed as proposed: card 9 gains a row in which a memory-pressure stop on the batch gate halts the leader, gates no prefix and sets no outcome.
- coverage 9 — MEDIUM — fixed as proposed: card 8 gains a row in which a flaky batch gate lands the batch and the leader's landing message carries no flaky section.
- coverage 10 — MEDIUM — fixed as proposed: card 8 gains a row in which a `failed` leader with live entries waiting lands alone and leaves them with no outcome.
- coverage 11, row-validity 6 — MEDIUM — fixed as proposed: card 6 gains a row in which a local-layer `finalize_lost_races` is refused. The `RetiredKeys` entry alone makes that true, so the row pins the entry. The default layer's warning itself is configengine's, pinned by its layer tests.
- row-validity 1 — MEDIUM — fixed as proposed: card 5 lists `tipRecordingResolver` under `Edit`, with a new `onResolve` hook that moves the parent during the lock-free merge. The moved-parent row now asserts that the merge under the lock brings in `p2`.
- row-validity 2 — MEDIUM — disputed, every fix worse: an `Edit` keeps the name `TestFinalize_GatesWithNoLockAndLocksToLand`, which D1 makes false, and a `Rename` pair raises `rename-tool-missing`, since `lym config language --print` shows no `rename_tools`. The delete-and-create on card 5 leaves the same code as a hand rename.
- row-validity 4, guard-reach 6, way-forward 2 — MEDIUM — fixed as proposed: card 4 names the input, a marker naming a deleted branch, and adds rows to `TestParentHoldsTaskWork_ReadsTheBranchWhileDetached` and `TestRestoreDetached`. Both errors end in `detachWayForward`, which names the marker path, the checkout and `lym loom resume`.
- row-validity 5, guard-reach 2 — MEDIUM — fixed as proposed: card 3's `MergesClean` and `BuildLandingChain` resolve refs first, as `HoldsWork` does. Card 3 adds a tier1 `TestMergesClean` test that passes branch names, a detached-head row for `MoveCodeTo` and a row for an unresolvable task head.
- row-validity 8 — MEDIUM — fixed differently: card 7's `actOnOutcome` drops the claim that its `landed` branch catches a moved task tip, since `Join` clears that outcome first (card 1). The row is kept, because it pins the criterion end to end through `Join`.
- guard-reach 3 — MEDIUM — fixed as proposed: card 8's `takeBatch` makes a run with no queue entry lead no batch, with a row for it.
- way-forward 3 — MEDIUM — fixed as proposed: card 10 wraps a restore error in the new `ErrRestoreRefused`, keeping the hook's way forward last, as `ErrLockedCheckRefused` does.
- way-forward 4 — MEDIUM — fixed differently: the proposal moves both stops into the shed restore for every row. That would replace Finalize's Stuck, which D5's criterion asks for. Card 10's `resumeRestore` instead refuses a dirty or unmarked detached worktree with D5's way forward at every row but Finalize, and Finalize keeps its own Stuck.
- way-forward 5 — MEDIUM — fixed as proposed: card 3's `MoveCodeTo` refuses dirt and a detached head with a `*MergeGuardError`. Card 8 removes the mark and ends Stuck with `dirtyParentReason`'s wording, and gains a row in which a parent dirtied during the gate is left unmoved with no mark.
- coverage 12 — LOW — disputed, every fix worse: a leader's darn tree scope is built by `internal/loomcli#darnTreeScope` (`internal/loomcli/darnwiring.go:88`), which is unexported and unreachable from `internal/landingshed`'s tests. The row therefore sits in loomcli, as a row of `TestDarnTreeScope_ClassifiesAtEachCall`.
- coverage 13 — LOW — fixed as proposed: card 8 gains a row in which a reset parent's `landed` run leads a batch that lands both runs.
- coverage 14 — LOW — fixed as proposed: card 7 gains a row for the parent-head fallback, a squash on the parent that no trailer names.
- coverage 15, row-validity 16 — LOW — fixed as proposed: rows are added for each branch the findings listed:
  - card 1: a member with no entry;
  - card 3: the 501-commit walk cap, the unresolvable head and the detached head;
  - card 4: `CheckoutBranch`'s dirt and the deleted-branch marker;
  - card 7: the mode read failure, the `Join` failure and the `Remove` failure;
  - card 8: the `MemberDeps` failure, a dead owner, `LandBlockers` and `ParentOwner`;
  - card 9: the marker standing at every step, and no gate on the merged culprit tree;
  - card 11: the backstop error.
  
  The card 10 restore branch of `detachedStop` is dropped, under guard-reach 5.
- coverage 16, forced-edits 4, row-validity 15 — LOW — fixed differently: no re-signed `runFixer` and no new entry point. `runFixer` already runs no gate before its first turn (`internal/landingshed/finalize.go:1221-1252`), so card 9's `failAsCulprit` writes the failure record and calls it as `gateAndFix` does. Card 9 declares the `finalizingSeats` fake and adds a gate-run count row.
- row-validity 9 — LOW — fixed as proposed: card 7 declares the `lockReadingResolver` fake.
- row-validity 10 — LOW — fixed as proposed: card 7's trail row sets a record path and lock under `t.TempDir()`, and a separate row covers the failed mode read.
- row-validity 11 — LOW — fixed as proposed: card 7's backstop row now asserts only that `Call` returns the error and runs no merge.
- row-validity 12 — LOW — fixed as proposed for two of its three points.
  - Card 8 names the mechanism that moves the parent mid-gate, a gate command that commits on the parent pair, and the `WriteLedgerLanding` test closure.
  - The member-commit point is disputed, premise false: card 7's row seeds a `landed` outcome by hand. Only the batch row reaches a member whose landed size is measured from a chain commit's first parent, another member's landing commit, through `recordLanding`'s new base.
- row-validity 13 — LOW — fixed as proposed: the darn scope case is now a row of `TestDarnTreeScope_ClassifiesAtEachCall`.
- row-validity 14 — LOW — fixed as proposed: the shed restore cases are now rows of `TestLockedCheck_RefusesUnderTheRunLock`.
- guard-reach 4 — LOW — fixed as proposed: card 7 names the input that reaches the `landed`-held branch, a later parent commit that rewrites a file the run changed, and adds its row.
- guard-reach 5 — LOW — fixed as proposed: card 10's `detachedStop` restore branch becomes a fail-loud wiring error naming `Shed.Restore`.
- guard-reach 7, way-forward 8 — LOW — fixed as proposed: card 11's backstop error names the mark file and the cause, with a way forward that keeps the mark, and gains a row.
- way-forward 6 — LOW — fixed as proposed: card 8 removes the mark on a false move, and card 1 states that `WriteMark` replaces a standing mark, with a row for it.
- way-forward 7 — LOW — fixed as proposed: on a failed restore, card 8 keeps the marker and ends Stuck with the dirty-path way forward that card 10's restore completes.
- forced-edits 3 — LOW — fixed as proposed: card 5's Intent states the rewrite of `finalize.go`'s lines 1-4 file comment.
- forced-edits 5 — LOW — fixed as proposed: card 7 re-signs `recordLanding`, `landedDiffEmpty` and `recordLandedSize` with the landed diff's base, and card 8 uses them.
- forced-edits 6 — LOW — the two parts get different dispositions:
  - Doc part, fixed as proposed: card 11 now lists `internal/fabriccli/doc.go` under `Extends`, with a sentence on the backstop.
  - Isolation part, disputed, premise false: `pattern/PATTERN-fabric-boundary.md`'s isolation rule bars knowledge of IDEs, shuttle, the board and heddle, and the landing queue and runpusher are none of them. D6 names `anchordeps.go`'s Acquire as a taker that runs the backstop.
