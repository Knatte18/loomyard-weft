```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind F1 and F2 and check that the fix covers every hit.

Unchecked findings: none from the review. F3 and F4 have no set as their premise, so neither needed a search.

No findings.

- **F1, non-Go files naming removed names:** I reran the fixer's search for the `Merge-In`, `merge-in:`, `gates.merge-in` and `MergeIn` spellings in non-Go files outside `_lyx`, `docs/research` and `docs/benchmarks`. Every hit is a target of card 15, 16, 17 or 18, except two:
  - `contracts/stencils/loom/loom-template-alignment.md`, which card 5 deletes.
  - `mill-config.yaml` lines 189, 197, 217 and 220. These are comments on the separate `mill-merge-in` tool and name neither loom's row nor its gate key.
  The fixer's case-insensitive search for alignment and parent-review names was already complete. Its leftover hits on `docs/overview.md`'s waiting-note sentence and the loom config and reconcile paragraphs are now covered by the card 4 and card 6 edits.
- **F2, test literals `Merge-In` and `merge-in`:** I reran the search over every `_test.go` under `internal`, without the quote-adjacent patterns, mapping each hit to its enclosing declaration. I left out fabric's own tests and the renamed `mergein` files, which the Intent now names. Every hit not named in card 16 falls into one of three groups:
  - **Git or fabric merge-in prose:** websterengine `TestParentMergeBetweenForkCommitAndRecordBatch`, `TestRecoveryRetry`, `TestRecordBatch_ParentMovedHead`, `TestRepositoryProbes` and `TestRefuseMidMerge_LinkedWorktree`; battenshed `TestWorktreeTeardown_RemoveFailureNamesRemovalHalf`; hubreconcile `markMergeRecord`; landingshed `TestFinalize_*`, `TestGuardedRows_LockLifecycle`, `TestVerifyGate_Clean`, `TestFinalizeVerify_*`, `newMergeInDepsAt` and `TestVerifyGate_RealVerifytreeScenario`.
  - **Free-form row-name fixtures, the kind the review rejected:**
    - verifytree `TestLandingFailureRecord`'s `row: Merge-In` in a `LandingFailure` record.
    - shedverbs `TestStatusCmd_FoundEnvelope_GenericCore`'s reroute-cap sentence.
    - landingshed `newFixerInputs`' `ReportPath` `/x/merge-in-fixer-report.md`, an arbitrary path.
  - **Already listed:** landingshed `fixerrun_test.go:87` `fixerTestRow`. My declaration mapper attributed it to `const (`, but the card names it.

Tool calls beyond context:
1. A non-Go search for `Merge-In`, `merge-in:`, `gates.merge-in` and `MergeIn`, against the target lists of cards 15–18. My context held only the fixer's search for alignment and parent-review names.
2. `mill-config.yaml`'s matching lines and its last commit, to classify the one non-Go hit no card targets.
3. A broad test-file search for `Merge-In` and `merge-in` mapped to enclosing declarations and filtered against card 16. The fixer's search needed a quote or colon next to the literal, so it could miss a literal embedded in a longer string.
