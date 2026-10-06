---
verdict: BLOCKING
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-uses-later-target.md
    summary: Card 1 edits the plan-review rubric stencil, which no Decision or Constraint licenses
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-uses-later-target.md
    summary: Card 1 un-pins check tallies in validate.go only, leaving the same tally pinned in planparser doc.go, normalize.go and the plan spec
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-uses-later-target.md
    summary: Card 1 leaves its fixture reorder to an unnamed grep although a known fixture in websterengine rebaseline_test.go would be refused by the new check
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/03-delete-target-gone.md
    summary: Cards 1, 2 and 3 each add a separate cobra validate drive to cli_test.go that repeats wiring the existing validate tests pin
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/plan/03-delete-target-gone.md
    summary: Card 3 leaves open whether delete-target-gone is named in the spec paragraph, although a new check owes its spec mention
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lyx/plan/04-drop-model-assertion.md
    summary: Card 4 removes the RoleMaster lookup but does not delete TestBeginBatch_UnknownRoleErrors, which pins it
  - id: F7
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/06-rebaseline-terminal-batch.md
    summary: Card 6 plans a recoverbatch_test.go test for a dead batch's live strand being stopped, which TestRecoverBatch_FirstCallSpawnsArchivesStaleReportAndStopsLiveStrand already covers
  - id: F8
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/08-background-shell-outcome.md
    summary: Card 8 says a stuck outcome leaves Master's strand alive until the next run, but a stuck outcome is a shuttle done outcome and the strand is removed
  - id: F9
    severity: MEDIUM
    class: design
    location: _lyx/plan/08-background-shell-outcome.md
    summary: Card 8 leaves the warning appended before the verify-gate demotion and only on a done outcome, so it cannot carry the same outcome wording as the note
  - id: F10
    severity: MEDIUM
    class: design
    location: _lyx/plan/09-gitrepo-leased-update.md
    summary: Card 9 builds CommitsNotIn as a go-git read like IsAncestor, but IsAncestor is a CLI call and the package doc records ancestry walks as measured and reverted from go-git
  - id: F11
    severity: LOW
    class: design
    location: _lyx/plan/10-reset-moves-remote-branch.md
    summary: RemoteOnlyCommits is a public Fabric method reused by webster reset and by Publish yet is created inside the reset card instead of a card of its own
  - id: F12
    severity: BLOCKING
    class: scope
    location: _lyx/plan/10-reset-moves-remote-branch.md
    summary: Card 10 adds the refusal for a checkout rewrite that fails after the remote moved, with a spec row, but plans no test that reaches it
  - id: F13
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-reset-moves-remote-branch.md
    summary: Card 10 does not say what envelope reset emits when the checkout fails after remote_branch_updated was recorded, so the recorded mutation would be dropped
  - id: F14
    severity: LOW
    class: consistency
    location: _lyx/plan/10-reset-moves-remote-branch.md
    summary: Card 10 leaves websterengine doc.go's reset paragraph and fabricengine doc.go's closed kind list describing a reset with no remote update
  - id: F15
    severity: LOW
    class: design
    location: _lyx/plan/00-overview.md
    summary: The verify section never runs the tmux tier of internal/loomcli, which card 11 targets
---

### [MEDIUM] Card 1 edits the plan-review rubric stencil with no licence

**Location:** `_lyx/plan/01-uses-later-target.md`, the `Edit:` list and the paragraph beginning "In `contracts/stencils/loom/loom-rubric-plan-review.md`".
**Issue:** The decision record's constraint names two homes for a new plan check: a row in `loom-plan-spec.md`'s "Validation checks" list and a row in the owning package's `doc.go` check list.
No Decision, Constraint or Scope item touches `contracts/stencils/loom/loom-rubric-plan-review.md`, a stencil that steers every Plan-Review round.
The edit is scope the answer key does not license.
**Fix:** Drop the rubric from card 1's `Edit:` list and its Intent.

### [LOW] Card 1's tally sweep is incomplete

**Location:** `_lyx/plan/01-uses-later-target.md`, the paragraph beginning "Update `validate.go`'s file comment".
**Issue:** The card un-pins "twenty-seven"/"twenty-eight" in `validate.go` only.
The same tally is pinned in `internal/planparser/doc.go` ("The plan format's 28 validation checks"), in `internal/planparser/normalize.go` ("through all twenty-eight pure checks") and in `contracts/specs/loom-plan-spec.md` ("through all twenty-eight checks", in the shape-classifier section).
After the new check lands each of those states a false count.
`doc.go` is the owning package's doc, which the record's constraint names.
**Fix:** Extend the sweep to those three places and add `internal/planparser/doc.go` and `internal/planparser/normalize.go` to the `Edit:` list; the spec is already a target.

### [MEDIUM] Card 1's fixture reorder is unnamed although a fixture is known

**Location:** `_lyx/plan/01-uses-later-target.md`, the paragraph beginning "Grep the existing planparser fixtures".
**Issue:** The card sends the implementer to grep for fixtures that `Uses:` a later card's target and reorder them, in files it does not list.
At least one exists: `handlePlan` in `internal/websterengine/rebaseline_test.go` has card 1 `Uses:` the handle that card 2 declares, and the arrow-shaped Create bullet puts that handle in card 2's `Targets`, so `uses-later-target` refuses it wherever the format set runs.
An unnamed edit leaves the card's target list short of the files it changes.
**Fix:** Name `internal/websterengine/rebaseline_test.go` in card 1's `Edit:` list with the `handlePlan` reorder, and keep the grep for any other fixture.

### [LOW] Three separate cobra validate drives repeat existing wiring

**Location:** `_lyx/plan/03-delete-target-gone.md`, with the matching paragraphs in cards 1 and 2.
**Issue:** Each of cards 1, 2 and 3 adds its own function to `internal/webstercli/cli_test.go` that seeds a plan, runs `c.validateCmd()` and checks the exit code and a check ID.
`TestValidateCmd_FindingsUseCardKey`, `TestValidateCmd_BlockingFindingCarriesSeverity` and `TestValidateCmd_ScopeFollowsRunProgress` already pin that wiring: findings keyed by card, severity on a blocking finding, and the whole-plan versus pending scope.
The decision record still requires each fix to be driven through the CLI once, so a drive per check stays; three items of one kind belong in one table-driven test (`PATTERN-test-economy`).
**Fix:** Card 1 creates one table-driven `TestValidateCmd_PlanGateChecks` in `cli_test.go`; cards 2 and 3 each add a row to it.

### [LOW] Card 3 leaves the spec mention of `delete-target-gone` open

**Location:** `_lyx/plan/03-delete-target-gone.md`, the paragraph beginning "Add `delete-target-gone` (informational)".
**Issue:** The card adds the ID to `doc.go` and to the spec paragraph "only if that paragraph enumerates resolve-backed IDs".
Card 2 creates that paragraph, so the condition is the implementer's call, which makes `delete-target-gone` and `delete-before-reference` differ in spec visibility for no stated reason.
**Fix:** Make it definite: card 3 adds `delete-target-gone` to the paragraph card 2 adds.

### [MEDIUM] Card 4 leaves the test that pins the removed role lookup

**Location:** `_lyx/plan/04-drop-model-assertion.md`, the `Tests:` paragraph.
**Issue:** The card removes "the RoleMaster lookup that feeds" the injection (`deps.Roles[target]` in `BeginBatch`).
`TestBeginBatch_UnknownRoleErrors` in `internal/websterengine/beginbatch_test.go` asserts that `BeginBatch` fails naming `RoleMaster` once `Roles` lacks it.
After the removal that behavior is gone and the test fails, but the card deletes only three other tests.
**Fix:** Add `TestBeginBatch_UnknownRoleErrors` to the deleted tests.

### [BLOCKING] Card 6 plans a test an existing one covers

**Location:** `_lyx/plan/06-rebaseline-terminal-batch.md`, the `recoverbatch_test.go` entry in `Tests:` and the `Edit:` list.
**Issue:** The card plans, in `recoverbatch_test.go`, a dead batch with a recorded live strand where `RecoverSpawnOrAttach` stops that strand before it starts the new one.
`TestRecoverBatch_FirstCallSpawnsArchivesStaleReportAndStopsLiveStrand` already seeds a `Terminal` batch with `Status: "dead"` and a live `StrandGUID`, and asserts the strand is removed before the fresh spawn.
The other half, that the recovery prompt carries the edited card, is asserted by the card's own `verbs_test.go` chain (rebaseline, then `recover-batch` spawning with the edited card in its prompt).
This breaks `PATTERN-test-economy`.
**Fix:** Remove `internal/websterengine/recoverbatch_test.go` from card 6's `Edit:` list and delete its `Tests:` entry.

### [MEDIUM] Card 8's "what ends it" is wrong for a stuck outcome

**Location:** `_lyx/plan/08-background-shell-outcome.md`, the paragraph beginning "What ends it is established by the code".
**Issue:** The card says that on a done outcome shuttle removes the strand and that on any other outcome, stuck included, the strand stays until the next `lyx webster run` reclaims it.
`Run.finalize` in `internal/shuttleengine/wait.go` removes the strand when the shuttle outcome is `OutcomeDone` and `KeepPane` is unset.
Master's own `stuck` and the verify gate's demotion are both shuttle-done outcomes, since `mapMasterDone` runs only under `OutcomeDone`, so both end the session.
Only the error outcomes (asking, died, timeout) leave the strand for the next run.
**Fix:** Key the clause on the shuttle outcome: done or stuck, whether Master's own or the gate's demotion, means the session ended and removed the shell with it; an error outcome means the strand stays until the next `lyx webster run`.

### [MEDIUM] Card 8's warning cannot carry the note's outcome wording

**Location:** `_lyx/plan/08-background-shell-outcome.md`, the paragraph beginning "In `Run`, move".
**Issue:** `Run` appends `expiredShellWarning` before the verify-gate demotion and only when `runResult.Outcome == outcomeDone`.
The card moves the note's write to after the demotion but leaves the warning where it is, so a demoted run would warn "done" and the note would say "stuck", and a Master-stuck run would get no warning at all.
D9 requires the same wording to reach both.
**Fix:** State that the warnings are built after the demotion, from the same outcome description the note uses, and appended on every outcome that returns a `RunResult` (done and stuck).

### [MEDIUM] Card 9 describes `IsAncestor` wrongly and picks the wrong backend

**Location:** `_lyx/plan/09-gitrepo-leased-update.md`, the `CommitsNotIn` paragraph.
**Issue:** The card builds `CommitsNotIn` "as a go-git local read like `IsAncestor`, with the same retrying object lookup".
`IsAncestor` is a CLI call (`git merge-base --is-ancestor` through `runChecked`), `ancestry.go` has no object lookup, and `internal/gitrepo/doc.go` lists `IsAncestor` and `HasUnpushed` among the methods "measured and reverted from a go-git ancestry walk".
**Fix:** Build `CommitsNotIn` on `git rev-list` through `runChecked`, validating both SHAs as `IsAncestor` does, and add it and `UpdateRemoteBranchLeased` to `doc.go`'s list of CLI-bound methods.

### [LOW] `RemoteOnlyCommits` earns its own card

**Location:** `_lyx/plan/10-reset-moves-remote-branch.md`, the `Create:` list.
**Issue:** `Fabric.RemoteOnlyCommits` is an exported method read by `ResetPairWarp` and, through `internal/loomcli`, by Publish in `internal/landingshed`.
The rubric gives a symbol reused across packages its own card, and this one sits inside the reset card.
**Fix:** Move it to a new card numbered before the reset card, with a direct integration test of its three answers (tip and commits, branch absent, no remote); renumber the reset and Publish cards.

### [BLOCKING] Card 10's checkout-failed refusal has no test

**Location:** `_lyx/plan/10-reset-moves-remote-branch.md`, the `Tests:` paragraph.
**Issue:** The card adds the refusal for a checkout rewrite that fails after the remote moved, and a spec row "reset: checkout failed after the remote update".
The tests cover the ancestry, lease, dirty, network and skip cases but never this one, which the record's acceptance criteria also name ("after a failed checkout rewrite a re-run converges").
`PATTERN-refusal-way-forward` requires a test that reaches each new refusal.
**Fix:** Add a case to `resetpairwarp_integration_test.go`: force the checkout rewrite to fail after the remote update, assert the message states the remote was already updated and names the re-run, and assert a re-run converges.

### [MEDIUM] Card 10 drops the recorded mutation on a failed checkout

**Location:** `_lyx/plan/10-reset-moves-remote-branch.md`, the `internal/webstercli/reset.go` paragraph.
**Issue:** `reset.go` turns any `ResetPairWarp` error into a bare error envelope.
After the remote update succeeds and the checkout fails, `rec` holds a `remote_branch_updated` entry that the envelope would not report, and the error text would still start "refused".
`PATTERN-mutation-record` reserves the bare envelope for pre-flight failures.
**Fix:** State that when `rec` is non-empty at the error the envelope carries `mutations` and `partial: true` (and omits "refused"), with the integration test and the webstercli test asserting it.

### [LOW] Card 10 leaves two docs describing the old reset

**Location:** `_lyx/plan/10-reset-moves-remote-branch.md`, the "Docs:" paragraph.
**Issue:** `internal/websterengine/doc.go` describes the reset envelope as carrying "the `worktree_reset` entry" with `partial` "always false" and states the reset's bound without the remote update, and `internal/fabricengine/doc.go` lists the closed `Kind` vocabulary.
Both go stale; the first file is not a target.
**Fix:** Add `internal/websterengine/doc.go` to the `Edit:` list and name both passages in the Docs paragraph.

### [LOW] The verify section skips loomcli's tmux tier

**Location:** `_lyx/plan/00-overview.md`, the `## verify:` section.
**Issue:** Card 11 targets `internal/loomcli/landingdeps.go`, and `internal/loomcli` holds several `tmux`-tagged test files.
The section runs `-tags tmux` for `internal/webstercli` only, so those tests are not run.
**Fix:** Add `./internal/loomcli/...` to the `go test -tags tmux` line.

## Focus settlements

- **Cards 5 and 7 test economy:** card 5 extends `TestBeginBatch_PreExistingReportRefused` and card 7 extends `TestBatchFailedError_TextAndSentinel`; their remaining functions pin behavior no existing test pins, and the cobra drives are required by the record. Card 6 is F7.
- **Card 1's rubric edit:** F1.
- **Card 8 against D9:** the element set is complete, but the two defects above are F8 and F9.
- **Cards 6 and 7 against D4 and D6:** every variant the record names (plan-edit, `run --fresh`, `recover-batch` first) is carried by a card and a test. The `webster-template-master.md` edit in card 7 is licensed as the consumer of D4's changed way forward, and it follows the `plan_drifted` rung's precedent.
- **Attack-surface question:** the Intent of cards 3, 6 and 10 each states what the change lets through and what bounds it, in terms of the record's Bound paragraphs; no finding.
- **Granularity:** cards 9 and 11 stay as they are (two small primitives for one consumer; one Publish unit), card 4 is one removal across its surfaces, and card 10 is F11.
- **Verify coverage:** F15; every other targeted package, `contracts/...` and `tools/sandbox` included, has its tagged tests run.
