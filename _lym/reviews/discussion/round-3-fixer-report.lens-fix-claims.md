```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked every bound and fact the round-3 changes assert against the code at HEAD.

Claims checked:
- **D1, keep scan:** `scanKeeps` is in `cmd/testtiming/keep.go`. "Kept … classification unchanged" holds: `cmd/testtiming/redundancy.go:430-431` says "The keep reason changes the report only, never the classification", and line 469 sets `verdicts[i].keep`. Holds.
- **D5, `Fabric.Status` covers modified and untracked files:** it holds. `gitrepo.WorktreeChangedFiles` (`internal/gitrepo/worktree.go:24-37`) takes any path whose Staging or Worktree status is not Unmodified, and an untracked path counts.
- **D5, the HEAD SHA read is exempt:** this holds. `pattern/PATTERN-fabric-boundary.md` exempts "read-only verbs (SHA, `status --porcelain`)".
- **Constraints, the step's merge-base reads go through `internal/fabricengine`:** this holds. `parentMergeBase` (`internal/gatecli/scope.go:670-685`) calls `fabricengine.ReadOrigin`, `fabricengine.Open` and `fabric.MergeBase`.
- **D7 step 4, "a non-tier holder that `internal/gateslot`'s doc lists":** see the first finding below.
- **D7 rationale, the worst case is the budget plus the longer of one build or one test plus grace:** this holds against step 4 as rewritten, because builds and tests run serially and nothing starts once the budget is spent. It describes the record's own rules, not code.
- **D5 and D6, the exit record and the process-ID match:** these are new design, not claims about existing code. No fact to check.

Findings:

1. **LOW.** D7 step 4: "The step takes one gate slot as a non-tier holder that `internal/gateslot`'s doc lists".
   The doc does not list the non-tier holders.
   `internal/gateslot/doc.go:62` says only "The non-tier holders hold one slot and set their own Go flags through HolderEnv", and that section names none of them. `pattern/PATTERN-gate-slot.md` also says "the non-tier holders `internal/gateslot`'s doc lists".
   The record should say the step joins that set as a non-tier holder, rather than imply an existing list names it.

2. **LOW.** D7 step 4: it takes the slot as a non-tier holder, then runs each test "under the test's tier `env`".
   `internal/gateslot/doc.go:39` gives a non-tier holder's child `HolderEnv`, and gives `Lease.InheritEnv` only to a tier run. `doc.go:51-52`: `HolderEnv` sets GOFLAGS `-trimpath` and `-p`.
   The changed text does not say how a non-tier holder's `HolderEnv` combines with the tier's `env` from `gate.yaml`. A plan writer could take either environment.
   D5's verb has the same setup, but D5's slot text did not change this round.

Calls made:
- One Bash grep, for the `internal/gateslot` doc's holder text, `parentMergeBase`, the keep handling in `redundancy.go` and the `fabricengine` constructors. None of these were in my context.
- One Read of `internal/gateslot/doc.go` lines 55-74, to see whether the doc lists the non-tier holders. The grep showed only line 62.
