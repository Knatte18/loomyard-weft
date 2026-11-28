# Card 9 — origin-judge-core

**Create:**
- `plan:internal/fabriccore#FieldReconcileTargets` -> `const FieldReconcileTargets Field = "reconcile_targets"`
- `plan:internal/fabriccore#FieldMovedHeads` -> `const FieldMovedHeads Field = "moved_heads"`
- `plan:internal/fabriccore#FieldBranchFacts` -> `const FieldBranchFacts Field = "branch_facts"`
- `plan:internal/fabriccore#ReconcileFacts` -> `type ReconcileFacts struct`
- `plan:internal/fabriccore#BranchFacts` -> `type BranchFacts struct`
- `plan:internal/fabriccore#OriginJudge` -> `func OriginJudge(snapshot Snapshot) Answer`
- `plan:internal/fabriccore#JudgeBranch` -> `func JudgeBranch(facts BranchFacts) fabriclayout.BranchOutcome`
- `plan:internal/fabriccore#EntryFacts` -> `type EntryFacts struct`
- `plan:internal/fabriccore#ClassifyEntry` -> `func ClassifyEntry(f EntryFacts) fabriclayout.EntryState`
- `plan:internal/fabriccore#ClassifyEntries` -> `func ClassifyEntries(scope fabriclayout.ReconcileScope, recorded []string, branches []fabriclayout.BranchOutcome, owners, subjects map[string]string) []fabriclayout.EntryOutcome`
- `plan:internal/fabriccore#BranchStopWayForward` -> `func BranchStopWayForward(b fabriclayout.BranchOutcome, dirs VerbDirs, emptyMarkerRefusal string) string`
- `plan:internal/fabriccore_test#TestOriginJudge` -> `func TestOriginJudge(t *testing.T)`
- `plan:internal/fabriccore_test#TestClassifyEntries` -> `func TestClassifyEntries(t *testing.T)`
- `plan:internal/fabricengine#reconcileReader` -> `type reconcileReader struct`
- `plan:internal/fabricengine_test#TestReconcileOrigin_RefusesAnUnsuppliedSeam` -> `func TestReconcileOrigin_RefusesAnUnsuppliedSeam(t *testing.T)`
- `plan:internal/fabricengine#TestReconcileOrigin_Snapshot` -> `func TestReconcileOrigin_Snapshot(t *testing.T)`

**Edit:**
- `cmd/lym#remoteRefsLockFetchAllowlist`
- `internal/fabriccore#Snapshot`
- `internal/fabricengine#ReconcileOrigin`
- `internal/fabricengine#judgeAgainstOrigin`
- `internal/fabricengine#judgeBranch`
- `internal/fabricengine#branchStopWayForward`
- `internal/fabricengine#StopWayForwardForTest`
- `internal/fabricengine_test#TestReconcileOrigin_Judge`
- `internal/fabricengine_test#TestFailedPushRecord`

**Delete:**
- `internal/fabricengine#lsRemotePair`
- `internal/fabricengine#entryFacts`
- `internal/fabricengine#classifyEntry`
- `internal/fabricengine#classifyEntries`
- `internal/fabricengine#EntryFactsForTest`
- `internal/fabricengine#ClassifyEntryForTest`

**Extends:**
- `internal/fabriccore/doc.go`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#runRule`
- `internal/fabricengine#lsRemoteOnce`
- `internal/fabricengine#fetchMovedHeads`
- `plan:internal/fabricengine#refuseUnsuppliedSeams`
- `plan:internal/fabricengine#SetRunRecordPaths`
- `plan:internal/fabriccore#RefuseUnsupplied`
- `plan:internal/fabriccore#VerbDirs`

**Intent:**
`ReconcileOrigin`'s judging half runs as phases of one core rule, `OriginJudge`, through `runRule` (D2, D3).

Phases, in today's order:
1. `PhasePreLock` reads `FieldSeams` and `FieldReconcileTargets` (today's `originTargets`: targets, pairs, the two repository directories, an absent pair).
   `OriginJudge` refuses through `RefuseUnsupplied` when `SeamRunRecordPaths` is unsupplied, or `SeamRunLocks` for a moving hub scope, the one scope that takes the prime lock (a run scope never does, `movePrime`), before any lock or remote call.
   A run scope with no registered pair and no `PairMayBeAbsent` refuses as today, as a `PhasePreLock` read error.
2. `PhaseOrigin` is locked by the engine's phase lock, which takes both repositories' remote-refs locks (`HoldRemoteRefs`) and holds them until `runRule` returns, as `judgeAgainstOrigin` holds them today.
   It reads `FieldOriginRefs` through `lsRemoteOnce` with the options' deadline and call hook.
   A transport read failure is answered with `ActionWait` of `RetryInterval` and a re-read, while today's attempts remain (`ReconcileOptions.attempts`); after the last, the answer is done and the report is `Unreachable`, nothing judged.
   A non-transport failure is returned, nothing judged.
3. When some target's origin head differs from its tracking ref, the rule asks for `FieldMovedHeads`, which fetches exactly those heads (`fetchMovedHeads`, judge-only by SHA into no ref); a failed fetch is returned.
4. `FieldBranchFacts` reads, per target, what `judgeBranch` reads today: the local tip, the tracking reflog's rewrite verdict for a code branch, and the two ancestry answers.
   The options' verdict hook runs before each target's read, as before each verdict today.

`ReconcileFacts` (in `Snapshot.Reconcile`) holds the targets, pairs, scope, attempts, retry interval and per-target `BranchFacts`.
`JudgeBranch` is `judgeBranch`'s decision over `BranchFacts`, in today's order: no origin head is gone; equal tips are level; rewound wins over ahead for a code branch; behind is fast_forward; ahead; a records branch neither side contains is needs_replay; a rewritten code branch; else diverged.
`ClassifyEntry` and `ClassifyEntries` are today's `classifyEntry` and `classifyEntries` word for word, `EntryFacts` today's `entryFacts`.
`BranchStopWayForward` is `branchStopWayForward`'s text per state over `VerbDirs` (card 5); `emptyMarkerRefusal` is the text of `refuseEmptyOwnMarker` for an anchor stop with no anchor, which the engine reads from disk and passes, so the core reads nothing.
The engine's `branchStopWayForward` delegates with `verbDirsOf` and that text, so its callers in `originreconcilemoves.go` are unchanged.

`ReconcileOrigin` keeps its signature for its callers in thirteen packages; its report, its errors and its printed mutations are unchanged (D5).
The ended entries, the owners and the moves stay as today until cards 10 and 11.
`fetchMovedHeads` now runs inside the reconcile's moved-heads read, so `remoteRefsLockFetchAllowlist`'s reason for it becomes "runs inside the reconcile's moved-heads read, under the remote-refs locks its phase lock holds".

Tests (the commit message lists them):
- `TestOriginJudge` (core, tier0): the pre-lock refusal for each unsupplied seam, judge-only and a run scope needing no run lock (the D13 core row);
  the retry loop over transport read failures waiting once per interval and ending unreachable after the attempts, and `attempts` of 1 with no retry;
  a moved head asks for the fetch and an unmoved one does not;
  and `JudgeBranch` rows for every state, taken from `TestReconcileOrigin_Judge`'s verdict rows.
- `TestClassifyEntries` (core, tier0) takes the fifteen `classifyEntry` precedence rows of `TestReconcileOrigin_Judge`, the D3 stop and busy-own-records rows, and the owned_elsewhere way forward.
  `EntryFactsForTest` and `ClassifyEntryForTest` go with their last user.
- `TestFailedPushRecord`'s `StopWayForward_*` rows move to `TestFailedPushWayForward` (card 5's core test) as `BranchStopWayForward` rows, but `AnchorStoppedPairWeftSetsTheAnchorInTheAnchoredDir`, which pins the anchored directory's read; `StopWayForwardForTest` stays for it.
- `TestReconcileOrigin_Judge` keeps the rows that pin git or an interleaving:
  the remote-refs locks held across every call and verdict hook and freed on every return, the refused-connection retry restored inside the sleep seam, the non-repository origin error, the fetch failing after both ls-remote calls,
  the judge-only fetch leaving every ref unchanged, with Bolt left level so the kept half reads no ledger (its diverged-Bolt half moves in card 12), the rewound rows read through the tracking reflog, the call count of a moved-nothing origin, and the `PairMayBeAbsent` refusal;
  it deletes the precedence table, the diverged, rewritten and gone verdict rows and their way-forward assertions, held by the core rows above.
  Its rows that rest on run records or the board move in card 12.
- `TestReconcileOrigin_RefusesAnUnsuppliedSeam` is a tier1 test in `internal/fabricengine_test`: it builds a hub through `internal/hubforge` first, unsets the run-records paths seam through `SetRunRecordPaths(nil)`, restoring it on return, and does not call `t.Parallel`.
  A moving `ReconcileOrigin` then refuses naming the seam, its report holds no branch, no entry and no mutation, and no branch or tracking ref moved.
- `TestReconcileOrigin_Snapshot` is a tier1 test in package `fabricengine`, the reconcile flow's seam test (D6):
  on a hubforge pair whose code branch is behind origin it runs the judge through `runRule`, asserts the snapshot each phase read (the targets, origin's heads, the moved heads, the branch facts) and the phases in order, then the report's verdicts.
The core tests add about 2 seconds to one tier0 run.

**ImpactSummary:** ReconcileOrigin keeps its signature for its thirteen production callers; judging moves into the core with the remote-refs locks still taken by the engine; two test shims go with their last user.
