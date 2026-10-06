---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-reset-moves-remote-branch.md:50
    summary: Card 11 plans a test for a remote tip that moved between fetch and update but names no way to reach it, and export_test.go is not among its targets
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/05-archive-unrecorded-report.md:24
    summary: Card 5 says TestBeginBatch_WayForward_ReportExistsIsRecorded keeps pinning a recorded batch, but that test seeds no batch record and so flips from refused to archived
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/04-drop-model-assertion.md:30
    summary: Card 4 rewords step 66 of the master stencil but misses its line 191 claim that begin-batch injects model changes, and its grep terms cannot match that line
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/12-publish-push-rejected.md:18
    summary: The new RemoteOnlyCommits closure in loomcli landingdeps.go is exercised by no test, because the every-field guard only proves it is non-nil
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/00-overview.md:46
    summary: The cli-drive Shared Decision says Publish needs a GitHub endpoint, but the push-rejected halt happens before any GitHub call, so only the resume half needs one
---

## Focus directive verdicts

1. **Lease rejection versus other failures.**
   `ResetPairWarp` can tell them apart without matching stderr outside `internal/gitrepo`: card 9 returns the bare sentinel `ErrLeaseRejected` from `UpdateRemoteBranchLeased` and wraps every other failure, and card 11 maps only the sentinel to the moved-tip refusal.
   Reach by test: the unreachable-remote and `pre-receive` rows reach the fetch failure and the update failure, but nothing reaches the lease refusal (F1).
2. **Pinned lists and `verify:`.**
   Card 9 edits `cmd/lyx/gitrepoboundary_test.go` for both new `*Repo` methods, card 11 edits `cmd/lyx/destructiveguard_test.go` for the banned token and the `updateRemoteBranch` row, and card 10's `RemoteOnlyCommits` reuses existing `Fetch`, `RemoteURL`, `ResolveSHA`, `CurrentSHA` and adds no `r.run` site.
   `## verify:` runs `cmd/lyx` in both `go test ./...` and the integration line.
   Not a defect.
3. **Existing tests the new checks reject.**
   Card 1 names `handlePlan` in `rebaseline_test.go` and the two backward `addCardUses` tests in `runlevel_test.go`; the swap in `handlePlan` keeps the #330 scenario.
   I found no other fixture with a `Uses:` naming a later card's target, and no existing plan with a Delete card outside planparser's parse test.
   One existing test is missed on the D5 side (F2).
4. **Spec and `doc.go` rows.**
   Cards 1, 2 and 3 each edit the spec's validation-checks list and the owning `doc.go`; cards 5, 6, 7, 11 and 12 each edit `refusal-spec.md` and the owning `doc.go`.
   Not a defect.

### [MEDIUM] Lease-rejected reset refusal has no reachable test

**Location:** `_lyx/plan/11-reset-moves-remote-branch.md:50`
**Issue:** The decision record's acceptance criterion requires that a remote tip that moved after the fetch makes reset refuse with the merge way forward and change nothing.
Card 11 lists this as a row of the new table-driven test, "a remote tip moved between fetch and update refuses ... (the `ErrLeaseRejected` path)".
`ResetPairWarp` fetches, checks and updates in one call, and the fabric reset tests are in package `fabricengine_test`, so no test can move the remote between the fetch and the update, and the card names no injection.
For the checkout failure the card explicitly leaves the injection to the implementer; for this row it says nothing.
`internal/fabricengine/export_test.go` already holds this kind of seam (`DeleteArchivedWeftBranchForTest` drives `deleteRemoteBranch` with a chosen lease), but it is not in card 11's Edit list.
**Fix:** Add `internal/fabricengine/export_test.go` to card 11's Edit list and name a test driver there that runs `updateRemoteBranch` with a lease SHA that is stale against the origin; the row asserts the lease-rejected refusal, its way forward, an unchanged remote and no record entry.

### [MEDIUM] Card 5 misdescribes an existing test that D5 flips

**Location:** `_lyx/plan/05-archive-unrecorded-report.md:24`
**Issue:** Card 5 says `TestBeginBatch_WayForward_ReportExistsIsRecorded` "keeps pinning the way forward against a recorded batch".
That test seeds a report on `newBeginFixture`'s state, which has no batch record, and asserts the refusal naming `record-batch`.
After D5 that exact state archives the report and succeeds, so the test fails at card 5's own gate unless it is edited.
**Fix:** Edit the card so this test seeds a recorded, begun and not terminal batch beside the report, which keeps its record-batch way forward meaningful, and fold the rest of the recorded-state refusal assertions into the existing tables rather than adding functions.

### [LOW] Master stencil line 191 still says begin-batch injects model changes

**Location:** `_lyx/plan/04-drop-model-assertion.md:30`
**Issue:** `contracts/stencils/webster/webster-template-master.md` line 191 reads "NEVER use a `/model` switch yourself — model changes are injected by Go's own `begin-batch` call, never chosen by you."
After D10 `begin-batch` injects nothing, so the clause is false.
Card 4 rewords step 66 only, and its grep terms ("AssertedModel", "assertedModel", "Injector", "model switch") do not match this line, because it spells the phrase with a backtick between "model" and "switch".
`contracts/stencils/claims_test.go` pins the clause "NEVER use a `/model` switch", so the reword has to keep that clause.
**Fix:** Add line 191 to card 4's docs paragraph: drop the "injected by `begin-batch`" tail and keep the pinned clause, and add "injected" to the grep terms.

### [LOW] The new landing closure is untested beyond being non-nil

**Location:** `_lyx/plan/12-publish-push-rejected.md:18`
**Issue:** Card 12 fills `Deps.RemoteOnlyCommits` in `internal/loomcli/landingdeps.go` by opening the pair and calling `Fabric.RemoteOnlyCommits`, and says the every-field guard in `landingdeps_test.go` covers it.
That guard only proves the field is populated.
The landingshed integration test builds its own closure over fabric, so a wrong location or a wrong method in the production wiring is seen by no test, and the stuck reason would silently degrade to the "remote tip could not be read" branch.
`landingdeps_integration_test.go` already drives a real `landingDeps` closure over a hubforge pair for `MarkTaskDone`.
**Fix:** Add `internal/loomcli/landingdeps_integration_test.go` to card 12's Edit list and extend it with one case: on a hubforge pair with a remote-only commit on the task branch, `landingDeps(...).RemoteOnlyCommits()` returns that tip and that one commit.

### [LOW] The cli-drive rationale misstates why Publish is not driven through the CLI

**Location:** `_lyx/plan/00-overview.md:46`
**Issue:** The `cli-drive` Shared Decision says driving Publish through `lyx loom step` "would need a GitHub fake in loomcli's harness".
Publish's push-rejected halt is Step 5 and the GitHub client is built at Step 6, so the rejected half needs no GitHub fake; only the resume half, which reaches the GitHub step, does.
The decision record's acceptance line says each item is driven through the CLI at least once, so the recorded reason for the departure has to be true.
**Fix:** Reword the rationale: Publish's rejected and resumed halves are driven at the producer over a hubforge hub because the resumed half reaches the GitHub step, which only landingshed's integration tier fakes; the closure's real wiring is driven in loomcli's `landingdeps_integration_test.go`.
