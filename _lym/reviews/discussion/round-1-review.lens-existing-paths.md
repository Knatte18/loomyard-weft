```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the existing code paths that read or write the state each Decision introduces or changes.

**Paths traced, per decision**

- D1, the closed table in `internal/shedrecipe/entries_implementer.go:21`: survives. The table gains a key and `implementerEntry` passes the rest of the row config, `gates` included.
- D1, the clause in `contracts/specs/implementer-spec.md:35` that commits the run directory only at Done: survives, because D1 edits the spec.
- D8, the arbiter description built from the recipe name alone (`internal/loomcli/arbiterprompt.go:34`): survives. D8 states this as a change, and the function has no other input.
- D9, the per-batch commit through `fabricengine.CommitAnchoredPaths` (`internal/fabricengine/commitweftpaths.go:41,80`): survives. Only the listed paths are staged, so other uncommitted work is left alone.
- D9, the PR rework archive through the `ArchiveWebster` seam (`internal/loomcli/wiring.go:716`, `internal/loomshed/prrework.go:24,47`): **breaks**. Finding 1.
- D10, Describe reading `summary.md` (`internal/loomcli/wiring.go:925-942`, `internal/loomshed/generation.go:132-146`): **breaks**. Finding 1.
- D10, batten's watch bound on a verify-fixer strand (`internal/battencli/wire.go:785,796-806`): **misreads**. Finding 4.
- D13, the plan gate (`internal/loomshed/gates.go:254-276`) and the rubric note (`internal/loomshed/publishfailure.go:21`): survive, since D13 names them.
- D13, the renumber bound read (`internal/loomcli/renumber.go:102`): the read survives, but the lease that serialises it does not. Finding 2.
- D13, the conflict route claim (`internal/shedadapters/webster.go:111`, `websterengine.ClaimConflictRoute`): survives, because D13 moves the record and its route claim to one shared path.
- D13, `heldBackReason` (`internal/loomcli/decision.go:437-449`): **misreads under D12**. Finding 3.
- D13 and D7, the merge-in in-flight warning (`internal/fabriccli/merge_verbs.go:20-37`, `websterengine.RunInFlight` in `internal/websterengine/inflight.go:21`): **misreads**. Finding 5.
- D11, the other users of the shared pieces (`internal/loomshed/verifygate.go:333`, `internal/loomcli/landingdeps.go:802`, `internal/failuretrace/locations.go:38` all call `ParseVerifyFailures` or use `VerifyFailure`): survive if the move keeps these names. The tree does not settle that.

**Findings**

1. **MEDIUM · design · D10 and D9 (D13's reader list): Describe never reads Weaver's `summary.md`, live or archived.**
   - Describe's live run record is hard-wired to Webster's directory: `RunRecordPath: summaryparser.Path(websterGeom.WebsterDir)` (`internal/loomcli/wiring.go:941`).
   - Prior generations come from `ArchivedWebsterDirs`, which reads only `prior-generation/webster` (`internal/loomshed/generation.go:140`, `internal/loomshed/prrework.go:24`).
   - The rework seam archives only Webster's record: `ArchiveWebster` calls `websterengine.ArchiveRunRecord` (`internal/loomcli/wiring.go:716-718`).
   - D10 claims Describe carries Weaver's triage and fixes "as it does Webster's", and D9 claims rework archives `_lym/weaver/` "as it does `_lym/webster/`". D13's list of readers that switch to the active implementer leaves out Describe's live and prior readers and the archive layout. A Weaver run's `## Integration suite triage` and `## Verify gate fixes` would therefore never reach the change description.

2. **MEDIUM · design · D13, renumber bound: the renumber verbs serialise against begin-batch through Webster's state-mutation lease, and the record routes only the read.**
   - `renumber.go:62` takes `websterengine.AcquireStateMutation`.
   - Webster's begin and record take the same lease (`internal/webstercli/beginbatch.go:133`, `internal/webstercli/recordbatch.go:111`).
   - Weaver writes its state under its own run lock (D12) and names no mutation lease shared with renumber.
   - Counterexample: a renumber reads Weaver's last begun card while Weaver is beginning the next batch. The renumber passes its bound check and rewrites card numbers after Weaver has recorded that batch's card set (D3).

3. **MEDIUM · design · D13 decision-notice guard against D12 attach.**
   - `heldBackReason` holds the notice back from a halted run's implementer strand. Its stated reason is that "the fresh session the run's re-entry starts carries the decision" (`internal/loomcli/decision.go:437-449`).
   - D12 has re-entry attach to the live in-flight Weaver strand found through `FindRun` rather than start a fresh one.
   - So a decision recorded while the run is halted never reaches that strand: the notice was held back, and the attached strand's prompt predates the decision. D13 extends the guard to Weaver strands without saying what happens to this case.

4. **LOW · design · D10 fixer bound.**
   - PATTERN-gate-slot sends a cold fixer only through `gatefixer`.
   - Batten's watch bounds any strand in `gatefixer.RoleVerifyFixer` by `webster.yaml`'s `RecoveryTimeoutMin` (`internal/battencli/wire.go:796-806`).
   - A Weaver verify fixer is therefore watched under Webster's recovery timeout. D10 names neither the fixer's role nor its timeout, and `weaver.yaml` (D5) holds no timeout key.

5. **LOW · design · D7 and D13, merge-in in flight.**
   - `lym fabric merge-in` warns only when `websterengine.RunInFlight` is true (`internal/fabriccli/merge_verbs.go:20-37`). That check reads Webster's `state.json` and `outcome.yaml` (`internal/websterengine/inflight.go:15-35`).
   - A Weaver run mid-flight gets no warning, yet D7 re-checks a stopped batch against its recorded start commit, a range a merge-in commit then crosses.
   - D13 does not list this reader, and Weaver writes no `outcome.yaml` (D9) for such a check to key on.

**Tool calls**

1. Grepped for importers of `websterengine` and for `summary.md` readers, to find the Webster-state readers that D13 and the manifest left out.
2. Read Webster-state uses in `merge_verbs.go`, `verifygate.go`, `landingdeps.go`, battencli's `wire.go` and `failuretrace`, plus the Describe and `ArchiveWebster` wiring, to check the readers call 1 found.
3. Grepped for callers of `LastBegunCard`, `RunInFlight`, `AcquireStateMutation`, `RunActive` and the conflict-record functions, plus battencli's `fixerTimeout` body, to see who serialises on Webster's lease.
4. Read the callers of `fixerTimeout` and the `RunInFlight` definition, to confirm the batten watch bound and what the in-flight test keys on.
