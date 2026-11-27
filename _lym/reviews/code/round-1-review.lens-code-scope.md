```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the goal points and the added entry of the decision record.

**Per change: the decision it traces to**
- C1 (the recording board sync and orch notifier in `TestMain`, keyed by board dir, replacing the per-test swaps so tests can run in parallel): traces to G1, cutting the tier1 time. Parallelism is not among the methods G2 lists, but PATTERN-test-speed's "Tests run in parallel" asks for it, and it moves no test to another tier. No finding.
- C2 (the three stale-build rows merged into one walk): G2, merging table cases that walk the same path.
- C3 (`TestEnsure_BoardWithNoRemoteCommitsHubWideConfigAndWritesStamp` dropped and its assertions folded into `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile`): G2, deleting a repeated test.
- C4 (the `restoresBoard` failed-walk row dropped and folded into `runFailedWalkAfterAReinterpretation`, with the added Worktree, status, stamp and clean assertions): G2, plus the assertion edits that fold forces.
- C5 (the retry walks dropped: the `EnsureOrWarn` repair, the unreadable pair config, the non-mapping template base, and the second walk in `arrangeUnpushedHubWide`): G2, deleting repeats.
- C6 (`newPrimeOnlyHub` used where no pair is needed): G1 and G2, lighter fixtures.
- C7 (the two unpushed-hub-wide subtests split into two top-level tests): forced by C1, so the replayed case can run in parallel.
- C8 (the reinterpretation tests regrouped under `TestEnsure_Reinterpretation`, with the "a rewritten value is mailed" and "no value to carry" rows dropped): G2 and C1.
- C9 (`TestEnsure_PairRegisteredDuringTheWalkIsReconciledBeforeTheStamp` moved under the heddle-render test, with `ensureAndAssertRendered` and `renderedFiles` extracted): G1, and forced by C1, since the worktree seam is process-wide.
- C10 (file headers and test doc comments updated): forced by C1 to C9.
- C11 (commit `f076676a2`, which rewraps comments): forced by the comment rules for the C10 comments.

**Per decision: the changes that build it**
- G1, cutting the tier1 time at least in half, measured before and after: C1 to C9. The change description reports 19.55 s → 3.65 s for `internal/hubreconcile`.
- G2, cutting by removing and merging tests, never by moving them to another tier: C2 to C5, C6 and C8. No test changes tier.
- G3, keeping one test per behaviour a PATTERN or criterion names: C3 and C4 carry their assertions over. Whether the C5 drops keep every behaviour covered is the code-tests lens's question; nothing in scope says otherwise.
- S1, only these packages' test files plus helpers only they use: built. The diff touches only `internal/hubreconcile`'s three `_test.go` files, and the recorders sit in `testmain_integration_test.go`.
- S2, no production change: built, and the diff stat names no production file.
- S3, before and after times per package and the net test-line change in the change description: built, but the line figure is stale (F1).
- A1, the added entry that cuts only packages at 15 s or more: built. `internal/configengine` and `internal/configsync` are unchanged, and their tier0 times are reported.

**Findings**
- F1 — LOW, consistency, `_lym/landing/summary.md` "Net test-line change". S3 asks for the net test-line change, and the summary says "394 added, 450 removed, so 56 fewer lines". That is the count for commit `f2ebdb2c1` alone. The range's `git diff --stat 2326ef4da..HEAD` gives 395 insertions and 450 deletions, so 55 fewer lines. Commit `f076676a2` (10 insertions, 9 deletions) made the summary's figure obsolete. The evidence is the per-commit and range `--stat` output already in the shared context.

No unrequested-scope finding, and no decision left unbuilt.

Tool calls: none.
