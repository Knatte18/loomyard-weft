```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I re-ran the search for each finding whose premise is a set (F1, F3, F4).

**Findings**

- LOW, `internal/webstercli/doc.go:117`. The module doc says "The refusal carries `run_ended: true` and the way forward of ending the turn". After F1 that holds only for the free-lock refusal. A probe-error refusal carries no `run_ended` and its way forward is to re-run the verb. The sentence is stale but not wrong for the case it describes. Evidence: `ownership.go:62` sets `run_ended` only when `runEnded` is true, and `doc.go:117` was not touched.

**Sets checked, all covered**

- F1, callers of `orphanedMasterRefusal`: the only caller is `refuseOrphanedMaster` (`ownership.go:59`), and it handles the new second return value. The Master stencil line (`contracts/stencils/webster/webster-template-master.md:193`) keys on `run_ended`, which a probe error no longer sets.
- F3, users of the empty-Failures path: `writeFailingIdentities` has two call sites. The new `len(r.Failures) == 0` branch precedes the final one (`verifygatereport.go:268`). The hang branch (`:257`) also lists failures. `failHung` takes its failures from `parseVerifyFailures`, which the gate never nils, so it is never empty there. Nothing else renders "Failing identities".
- F4, files that list `notice_errors`: only `internal/loomcli/decision.go`, `docs/overview.md` and `decision_test.go`. The first two were updated, and `loomcli/doc.go` already carried `notice_held_back`. The test file is outside the finding.

**Tool calls**
1. `grep orphanedMasterRefusal` over `*.go`, to find callers other than `ownership.go`'s own definition.
2. `grep -l notice_errors` over the tree, to find every file listing the envelope keys.
3. `grep run_ended` outside tests, to find the stencil and docs that depend on that key.
4. `grep writeFailingIdentities|Failing identities` outside tests, to find every renderer of the identity list.
