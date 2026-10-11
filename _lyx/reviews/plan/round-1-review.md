---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/09-land-row.md:75
    summary: Card 9 adds the Land row and the Land registry entry but leaves untargeted the tests that pin the four-row recipe and the closed registry, so the untagged and integration tiers fail
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/00-overview.md:29
    summary: A pause honoured as a blocked Stuck in Land leaves pause_requested set, so the named way forward lyx batten run only records paused and the row is not re-run until a second run
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/08-land-producer.md:72
    summary: Several prescribed Stuck and refusal branches across cards 8, 9 and 10 have no named test that reaches them, among them the driver-busy resume refusal of the automatic Publish retry
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/11-orch-docs.md:15
    summary: Card 11 leaves the orch stencil's run-loop item 2, which says batten never resumes a halted child on its own, contradicting D5's automatic Publish retry
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/11-orch-docs.md:9
    summary: Card 11 targets contracts/stencils#stencilClaims, the row struct type, while the orch-template-role.md claims it edits live in the wordingClaims table
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/plan/02-fabric-push-retry.md:42
    summary: Cards 1 and 2 test the same retry loop and refusal classification twice, once as gitrepo's PushWithRetry table and again through fabric's TestPushBranchWithRetry, which gains a cannot-lock-ref row card 1 already holds
  - id: F7
    severity: LOW
    class: design
    location: _lyx/plan/05-landing-deploy-key.md:12
    summary: TestLoadConfig_Deploy is a per-key copy of TestLoadConfig_PublishVerify's key-set and key-absent rows instead of new rows in that test
  - id: F8
    severity: LOW
    class: design
    location: _lyx/plan/07-row-failure-notice.md:17
    summary: TestWire_RowNoticesReadBattensOwnHistory repeats TestWire_PauseRequestedReadsBattensOwnStatus's fixture for a second field of the same status file, while card 9's Land.PauseRequested wiring gets no assertion at all
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/10-publish-retry.md:68
    summary: Card 10 lists conditionText as edited but never says how a function of (finding, status) learns the retry is spent, so either its re-sign arrow is missing or the target is not needed
  - id: F10
    severity: LOW
    class: design
    location: _lyx/plan/09-land-row.md:70
    summary: Card 9 guards goto onto Land but does not state that the existing goto onto Worktree-Teardown with a done child now skips Land's pull, deploy, prune and ready notice
  - id: F11
    severity: LOW
    class: scope
    location: _lyx/plan/09-land-row.md:70
    summary: haltedOverDoneChild's status text still says lyx batten run runs Worktree-Teardown, while after card 9 it first runs Land
  - id: F12
    severity: LOW
    class: scope
    location: _lyx/plan/11-orch-docs.md:15
    summary: internal/orchcli/doc.go's run-loop step 5 still tells the orch to pull, deploy and reconcile after landing, and no card updates it
  - id: F13
    severity: LOW
    class: design
    location: _lyx/plan/01-gitrepo-push-retry.md:5
    summary: Card 1 exports RetryablePushRefusal although no card calls it outside internal/gitrepo
---

Generation 0: `_lyx/loom/rework/` does not exist, so the decision record is the whole answer key.
Fidelity holds: D1 to D9 each have a carrying card, D3 is honoured, since no card runs a reconcile, D6's exclusions are stated on card 1, D8's stencil edits and D9's `CLAUDE.md` and refusal-spec rows are on card 11, and every document in the Constraint list has a card.
No card adds scope the record does not license.
Card order is sound: every `plan:` handle a card uses is created by an earlier card.
`## verify:` runs `./...` on every tier, in the template's order, and no targeted package lies in a nested module.

### [BLOCKING] Card 9 leaves the tests pinning the four-row recipe and the closed registry untargeted

**Location:** `_lyx/plan/09-land-row.md:75`

**Issue:** Card 9 adds a fifth recipe row and a new registry key, `Land`.
It edits `TestNew_RowNamesMatchTheDurableIdentityConstants` and `TestNew_OnDoneChainAndLoadBearingEdges`, but these tests also pin the old shape and are on no card:
- `internal/battenrecipe#TestNameRunShed_ValueIsPinnedAgainstASymmetryRename` (`recipe_test.go:136`) asserts `len(shed.Producers) == 4` against a four-name `wantNames` list, so it fails on the untagged tier.
- `internal/shedrecipe#TestNames` (`registry_test.go:48`) asserts `Names()` equals an exact list without `Land`, so it fails untagged.
- `internal/shedrecipe#TestEveryEntry_RejectsUnknownConfigKey` calls `t.Fatalf("registry entry %q has no row in minimalEntries ...")` for any registered name `minimalEntries` (`conformance_test.go:19`) lacks, so it fails untagged.
- In the integration tier, `internal/battencli#fourRowRunForRecipe` steps the embedded recipe and expects its fourth step to be Worktree-Teardown and the pair gone.
  After card 9 that step is Land, which pulls the hub prime and leaves the pair in place.
  `stepAwaitingApprovalResumeDoneTeardown_ArchivesTheRunRecords` (`records_integration_test.go:32`) asserts that the step after Run-Shed's Done is Worktree-Teardown.
  `shortPollBattenRecipe` (`lifecycle_integration_test.go:64`) claims to be the embedded recipe "byte-for-byte the same four rows".

The plan's `## verify:` (`go test ./...` and `go test -tags integration ./...`) fails on these tests.
The card's ImpactSummary says every batten run passes through the new row, but the card does not follow that through to the tests that step through the recipe.

**Fix:** Add `internal/battenrecipe#TestNameRunShed_ValueIsPinnedAgainstASymmetryRename`, `internal/shedrecipe#TestNames` and `internal/shedrecipe#minimalEntries` to card 9's `**Edit:**`.
Also add the integration members `internal/battencli#fourRowRunForRecipe`, `internal/battencli#stepAwaitingApprovalResumeDoneTeardown_ArchivesTheRunRecords` and `internal/battencli#shortPollBattenRecipe`.
The Intent then says how each one changes: Land in the name lists, a Land row in `minimalEntries`, and the integration steps passing through Land, with the hub prime and board in a state Land completes on, before Worktree-Teardown.
Use member glyphs rather than the file paths, to pass `redundant-file-target`.

### [MEDIUM] A pause honoured in Land leaves `pause_requested` set, so the named resume only records paused

**Location:** `_lyx/plan/00-overview.md:29` (Shared Decision `land-row-pause`), `_lyx/plan/08-land-producer.md` step 1

**Issue:** The Shared Decision says a pause during the land-lock or dirty-prime wait "ends the wait with a Stuck that names the pause and `lyx batten run <slug>`, halting batten blocked at Land", and that "the resume re-runs the row from the top".
Shed clears `PauseRequested` only in the persist that records `StatePaused`, either at step 3 or in Goto (`internal/shedengine/run.go:215`, `persistTransient`'s `consumePause`, `goto.go:95`).
A Stuck with no `on_stuck` persists `blocked` and leaves the flag set, and no run or resume verb clears it (`internal/shedverbs/pause.go` is its only writer).
The first `lyx batten run <slug>` after the halt therefore reaches step 3 with `pause_requested` true, records `paused` and returns without calling Land.
Only a second `lyx batten run` re-runs the row.
The Stuck reason's way forward and the Shared Decision are both wrong about what the resume does.
Card 8's tests stop at "a pause during it is Stuck with no notice" and never drive the named way forward through shed.

**Fix:** Settle the pause outcome against shed's actual flag handling, and make the Stuck reason name a way forward that re-runs Land, or state the two-step resume truthfully in the Shared Decision, the reason and `doc.go`.
Add a test, a shed-level one over `battenrecipe.New` or a row in card 9's tests, that drives a pause during Land and then the named way forward, and asserts that Land runs again.

### [MEDIUM] Prescribed Stuck and refusal branches with no named test, across cards 8, 9 and 10

**Location:** `_lyx/plan/08-land-producer.md:62` and `:72`, `_lyx/plan/09-land-row.md:59`, `_lyx/plan/10-publish-retry.md:63` and `:81`

**Issue:** The same gap recurs in all three cards: a branch is prescribed in `**Intent:**`, and no named test reaches it.
- Card 8 makes "every failed step, a seam read error included" Stuck naming the step and sending one notice.
  Its failing-steps table names a non-zero deploy, a timed-out deploy, a deploy shell that cannot start, a pull refusal, a spent prime-lock wait, a board read and a prune failure.
  It names no `ParentBranch`, `PrimeBranch`, `PrimeDirty` or `Landing` read error, no `LandLock.Acquire` error, and no ready-snapshot marker read or write failure.
- Card 9's `ParentBranch` seam refuses an empty parent branch, with no input or test named.
- Card 10 prescribes a resume refused with `ErrChildDriverBusy` during the automatic retry, retried and sending the driver-busy notice through `noticeDriverBusy`, which D5 requires.
  Its tests name only "a goto refused as busy and a resume refused as not parked".
  Its `PublishFailure` and `ChildHead` seam errors, warned about and left to today's handling, also have no named test.

**Fix:** Add rows to the tests each card already plans: the seam-error rows to card 8's failing-steps table, an empty-parent-branch row in battencli for card 9's `ParentBranch`, and to card 10's tables a driver-busy resume row and a seam-error row for `PublishFailure` and `ChildHead`.
Each row asserts the Stuck, or the no-retry outcome, and the notice.
If a branch cannot be reached after its upstream checks, drop it from the Intent.

### [MEDIUM] The orch stencil's run loop still says batten never resumes a halted child

**Location:** `_lyx/plan/11-orch-docs.md:15`; `contracts/stencils/orch/orch-template-role.md:26`

**Issue:** Item 2 of `## The run loop` reads "A halted child (blocked, paused or failed) is a wait batten never resumes on its own".
D5 makes the first Publish verify failure an exception, and card 10 updates battenshed's `doc.go` for the same rule.
Card 11 edits `## After landing`, `## PR-Gate`, `## Where a finding goes` and `## Escalations from a child`, but leaves this item unchanged.
The stencil the orch reads then contradicts the Escalations line card 11 adds.

**Fix:** Have card 11 reword run-loop item 2 to name the one exception: batten sends the first Publish verify failure of a run back to the record's target itself.
Add a `stencilClaims` claim for it if the row's other claims pin the wording.

### [MEDIUM] Card 11 targets the claim struct type, not the claim table

**Location:** `_lyx/plan/11-orch-docs.md:9`

**Issue:** `contracts/stencils#stencilClaims` is `type stencilClaims struct` (`claims_test.go:87`).
The `orch-template-role.md` row whose claims card 11 drops and adds is an element of `var wordingClaims = []stencilClaims{` (`claims_test.go:107`, the row at `:655`).
The card's Edit target names a member the card does not change, and omits the one it does.

**Fix:** Replace the target with `contracts/stencils#wordingClaims`, and say "`wordingClaims`' row for `orch-template-role.md`" in the Intent.

### [MEDIUM] The push retry loop is tested twice

**Location:** `_lyx/plan/01-gitrepo-push-retry.md:41`, `_lyx/plan/02-fabric-push-retry.md:42`

**Issue:** Card 1 creates a `PushWithRetry` table covering success first time, a refusal then success with one sleep, a persistent refusal stopping at `pushMaxAttempts` with doubling sleeps, and a non-retryable error returned after one push.
It also creates a `RetryablePushRefusal` table that includes a remote `cannot lock ref` row.
After card 2, `pushSeam.pushBranchWithRetry` hands its loop to `gitrepo.PushWithRetry`.
Yet `TestPushBranchWithRetry_RetriesOnlyTransientRefusalsUpToTheBound` keeps its loop rows: success first try, refusal then success, persistent refusal at the bound with doubling sleeps, hook declined attempted once, and non-fast-forward attempted once.
Card 2 also adds a `cannot lock ref` row to it.
Each of those rows now re-tests card 1's `PushWithRetry` and `RetryablePushRefusal` behaviour through a thin wrapper (`PATTERN-test-economy`).

**Fix:** Cut fabric's test to what only the wrapper does: `git push -u origin <branch>` in the given dir with `branch` as the ref, and the seam's sleep passed through.
One persistent-refusal row asserting `errors.Is(err, gitrepo.ErrPushRetriesExhausted)` is enough, and the `cannot lock ref` row is dropped.

### [LOW] `TestLoadConfig_Deploy` duplicates `TestLoadConfig_PublishVerify`

**Location:** `_lyx/plan/05-landing-deploy-key.md:12`

**Issue:** `TestLoadConfig_PublishVerify` (`internal/landingshed/config_test.go:111`) already pins that an optional string landing key loads as written, and that a file without it loads the template's empty value.
The new `TestLoadConfig_Deploy` repeats that shape, key set and key absent, for a second key: one test per symbol.

**Fix:** Make the existing test a table over the optional string keys, with `deploy` rows beside the `publish_verify` ones, by targeting `internal/landingshed#TestLoadConfig_PublishVerify`.
Drop the `TestLoadConfig_Deploy` create and the `internal/landingshed` budget raise.

### [LOW] Batten-status seam tests split per field, and Land's pause seam untested

**Location:** `_lyx/plan/07-row-failure-notice.md:17`, `_lyx/plan/09-land-row.md` Wiring

**Issue:** `TestWire_RowNoticesReadBattensOwnHistory` writes batten's own status file and asserts a value read from it, and 0 with the file absent.
That is `TestWire_PauseRequestedReadsBattensOwnStatus`'s fixture and absent-file row (`internal/battencli/wire_seams_test.go:174`), applied to a second field.
Card 9 then wires `Land.PauseRequested` to "batten's own status as Run-Shed's does", and no test asserts it.

**Fix:** Extend `internal/battencli#TestWire_PauseRequestedReadsBattensOwnStatus` into a table over the seams that read batten's own status: `InnerRun.PauseRequested`, `Land.PauseRequested` and `Teardown.Notices.BattenHistoryLen`.
Drop the separate create and its budget raise.

### [LOW] `conditionText`'s edit is unspecified

**Location:** `_lyx/plan/10-publish-retry.md:23` and `:68`

**Issue:** `conditionText(finding noticeFinding, status shedengine.Status) string` has no input that says whether the publish retry is spent.
The card says only that it "adds `automatic Publish verify retry already ran`".
Either the function gains a parameter, and its Edit bullet lacks the re-sign arrow, or `noticeCondition` puts the text into `noticeFinding.text`, which `conditionText` already renders, and `conditionText` needs no edit.

**Fix:** State which.
For a new parameter, write `` `internal/battenshed#conditionText` -> `<new head>` `` and list its callers.
Otherwise drop `conditionText` from the targets and say `noticeCondition` sets the finding's text.

### [LOW] The goto onto Worktree-Teardown now bypasses Land, unstated

**Location:** `_lyx/plan/09-land-row.md:70`

**Issue:** `preGoto` admits a goto onto Worktree-Teardown whenever the child's status reads done (`internal/battencli/goto.go`).
After card 9, a batten blocked at Land has a done child.
`lyx batten goto <slug> --to Worktree-Teardown` therefore tears the pair down with no pull, deploy, prune or ready notice.
Card 9 bounds the new edge onto Land, but never states what the existing edge can now skip.

**Fix:** In card 9's Intent, state that the Worktree-Teardown goto skips Land and whether that is kept as the operator's escape hatch.
If it is not kept, tighten `preGoto` to admit Worktree-Teardown only once Land has returned Done, with a `TestPreGoto` row.

### [LOW] The status flag text still says the run goes straight to Worktree-Teardown

**Location:** `_lyx/plan/09-land-row.md:70`; `internal/battencli/arm.go:653`

**Issue:** `haltedOverDoneChild` renders "run \"lyx batten run <slug>\" from the prime, which runs Worktree-Teardown".
After card 9, that run starts at Land, or at whichever row batten halted at, and pulls, deploys and prunes before the teardown.

**Fix:** Add `internal/battencli#haltedOverDoneChild` and `internal/battencli#TestHaltedOverDoneChild` to card 9, with wording that names Land and Worktree-Teardown, or neither.

### [LOW] `internal/orchcli/doc.go` still describes the old landing routine

**Location:** `_lyx/plan/11-orch-docs.md:15`; `internal/orchcli/doc.go:38`

**Issue:** The package doc's run loop says: "After landing, from the prime: pull, run the production deploy, reconcile config ...".
The Goal moves those steps off the orch, and card 11 moves them out of the stencil, but this doc keeps them.

**Fix:** Add `internal/orchcli/doc.go` to card 11's targets, and reword step 5 to match the new `## After landing`.

### [LOW] `RetryablePushRefusal` is exported with no outside caller

**Location:** `_lyx/plan/01-gitrepo-push-retry.md:5`

**Issue:** Card 2 uses `gitrepo.PushWithRetry` and `gitrepo.ErrPushRetriesExhausted` only, and nothing outside `internal/gitrepo` calls `RetryablePushRefusal`.
Exporting it adds public surface no consumer needs.

**Fix:** Declare it unexported, as `plan:internal/gitrepo#retryablePushRefusal`.
Card 1's in-package test still reaches it.
