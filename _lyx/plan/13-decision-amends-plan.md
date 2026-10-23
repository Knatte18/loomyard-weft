# Card 13 — decision add --amends-plan

**Edit:**
- `internal/loomcli#decisionInput`
- `internal/loomcli#decisionDeps`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#decisionState.preRun`
- `internal/loomcli#loomCLI.decisionCmd`
- `internal/loomcli#discussionCommitPathspec`
- `internal/loomcli#decisionFake`
- `internal/loomcli#decisionFake.deps`
- `internal/loomcli#newDecisionFake`
- `internal/loomcli#TestDiscussionCommitPathspec_IncludesParentReviewDir`
- `internal/loomcli/doc.go`
- `docs/overview.md`

**Create:**
- `plan:internal/loomcli#decisionWayAmendsPlanRetry` -> `const decisionWayAmendsPlanRetry = ""`
- `plan:internal/loomcli#decisionWayFabricCommit` -> `const decisionWayFabricCommit = ""`
- `plan:internal/loomcli#decisionWayPlanUnparsed` -> `const decisionWayPlanUnparsed = ""`
- `plan:internal/loomcli#errPlanUnparsed` -> `var errPlanUnparsed error`
- `plan:internal/loomcli#amendmentPathspec` -> `func amendmentPathspec(location *lyxcwd.Location, tracked func(rel string) (bool, error)) []string`
- `plan:internal/loomcli#planApprovedReader` -> `func planApprovedReader(anchorPath string) func() (bool, error)`
- `plan:internal/loomcli#TestDecisionVerb_AmendsPlan` -> `func TestDecisionVerb_AmendsPlan(t *testing.T)`
- `plan:internal/loomcli#TestAmendmentPathspec` -> `func TestAmendmentPathspec(t *testing.T)`

**Uses:**
- `internal/loomshed#AppendAmendment`
- `internal/loomengine#LoomAmendmentRecord`
- `internal/loomengine#LoomAmendmentRecordRel`
- `internal/loomcli#committedHoldsEntry`
- `internal/loomcli#goodDecisionInput`

**Intent:**
D7: `lyx loom decision add` gains `--amends-plan`, a boolean flag; with it the verb also writes the amendment record, in the same commit as the entry.

`decisionInput` gains `amendsPlan`; `decisionCmd` registers the flag and its `Long` text says what it does, when it is refused and that the parent then runs `lyx loom goto --to Plan-Burler-Review` and `lyx loom resume`.
`decisionDeps` gains `amendmentPath`, `approvedPlan func() (bool, error)`, `appendAmendment func(title string, at time.Time) error`, `committedAmendment func() ([]byte, bool, error)` (the record as committed at HEAD) and `commitAmendment func() error` (a fabric commit of the record alone); `preRun` wires them over `loomengine.LoomAmendmentRecord(target)`, `planApprovedReader(target.AnchorPath())`, `loomshed.AppendAmendment`, `fabricengine.CommittedAnchoredFile` and `fabricengine.CommitAnchoredPaths` with `LoomAmendmentRecordRel()`.
`planApprovedReader` stats the plan directory's overview first and answers false when it is absent, since `ParsePlan` reports an absent overview without wrapping `fs.ErrNotExist`; it then parses the plan and answers its `approved:` flag.
A `ParsePlan` error holding an `*fs.PathError` (`errors.As`) is a read failure, returned as is; any other `ParsePlan` error is a parse failure, returned wrapping `errPlanUnparsed`; a failed stat other than not-exist is returned as is.

`decisionVerb` with `amendsPlan` refuses a darn run (no plan to amend) right after `deps.darn()` answers true and before `checkDarnRecord`, since `--amends-plan` has no plan to amend on any darn run; its way forward is to record the decision without the flag.
After its existing caller, form and darn checks it refuses when `approvedPlan` answers false (the current generation has no approved plan), naming the way forward: the decision is recorded without the flag, and Plan-Write and Plan-Review read the record.
An `approvedPlan` error wrapping `errPlanUnparsed` is refused with `decisionWayPlanUnparsed`, which names `lyx loom validate-plan` to show the defect and recording the decision without `--amends-plan` until the plan parses; any other `approvedPlan` error is refused with `decisionWayRetry`.
It reads the decision record's bytes and the amendment record's bytes, and whether it existed, before the call, then calls `appendAmendment(title, now)` before the append.
A decision record absent before the call is not refused there: the verb goes on to the append, whose existing `fs.ErrNotExist` case refuses with `decisionWayNoRecord` after restoring the amendment record; the decision record, absent before the call, stays absent, since its restore writes prior bytes only for a record that existed.
A failed read of either record, other than an absent file, is a propagated raw I/O error, refused with `decisionWayRetry`, and so is a failed `appendAmendment`, which writes nothing: the verb restores the amendment record to its prior bytes (removing it when this call created it) and has not touched the decision record.
When the append returns an error or check findings, it restores the amendment record to its prior bytes, removing it when this call created it; on an append error, which `AppendDecision` returns before its own restore can run, it also restores the decision record to its prior bytes.
When `committedAmendment` then answers this call's bytes, a status commit landed them, and the verb runs `commitAmendment` on the restored record before it refuses with the existing way forward.
When that `commitAmendment` fails, HEAD keeps a title whose entry never landed: the verb keeps the restored files and refuses with `decisionWayFabricCommit`, since `lyx fabric commit` stages `_lyx` whole and lands the restored amendment record, and the re-run of the verb comes after it.
When the commit fails, it reads HEAD: `committedAmendment`, and the decision record through `committedHoldsEntry`.
Equal to this call's bytes and holding the entry's heading: the commit landed; it keeps both files and succeeds, its envelope carrying `commit_error`.
Equal but without the heading: a status commit landed the record alone; it keeps both files and refuses with `decisionWayFabricCommit`, `lyx fabric commit`, which commits the entry.
Different: it restores both files to their prior bytes, removing the amendment record when it created it, and refuses with `decisionWayAmendsPlanRetry`, which re-runs `lyx loom decision add … --amends-plan` and never `lyx fabric commit`.
A HEAD it cannot read keeps both files and refuses with `decisionWayFabricCommit`, since a re-run over a landed commit would append a duplicate entry.
A call without the flag keeps today's behavior and way forward.
`decisionWayFabricCommit` names `lyx fabric commit` and keeps today's commit-failure clause that a session lyx refuses the verb from reports status: FAILED and the orch runs it, since the `--by arbiter` caller cannot run it (PATTERN-no-denied-recovery); `decisionWayAmendsPlanRetry` names the re-run of `lyx loom decision add … --amends-plan`.
Reaching inputs: each branch is a `decisionFake` state that passes every upstream check: an approved plan, a parent caller, a loom run; the darn refusal's fake is a darn run whose record check is never reached.

`discussionCommitPathspec` appends `amendmentPathspec(location, …)` over `fabricengine.PathTracked`, so the entry and the record land in one commit; `amendmentPathspec` returns `LoomAmendmentRecordRel()` when the record exists or `tracked` answers true, and nothing otherwise, a `tracked` error included, so git is never handed a pathspec matching nothing.
What the flag lets through: one entry and one title per call; the admission it enables is bounded on cards 14 and 15.

`internal/loomcli/doc.go` documents the flag, the record, the restore and commit rules; `docs/overview.md`'s `decision add` line names `--amends-plan`.

Tests:
`TestDecisionVerb_AmendsPlan`, over `decisionFake` with an amendment path in a temp dir and the new seams, rows: an approved plan lands the entry and a one-title record in the verb's one commit; a second call appends its title; a darn run and an unapproved plan are refused naming their way forward with no file changed; an `approvedPlan` error is refused with `decisionWayRetry`; an `approvedPlan` error wrapping `errPlanUnparsed` is refused with `decisionWayPlanUnparsed`; an absent decision record is refused with `decisionWayNoRecord`, leaving the amendment record as before and the decision record absent; a failing `appendAmendment` is refused with `decisionWayRetry`, leaving both records as before; an append refused with check findings leaves the decision record without the entry and the amendment record as before, so a failed second call keeps the first title; an append error restores both records; a findings refusal after a status commit landed the amendment bytes runs `commitAmendment`, and when that commit fails it refuses naming `lyx fabric commit`; a commit error with HEAD holding both succeeds with `commit_error`; a commit error with HEAD holding the bytes and no heading keeps both files and names `lyx fabric commit`; a commit error with HEAD differing restores both and names the `--amends-plan` re-run; an unreadable HEAD keeps both and names `lyx fabric commit`.
Since the verb's rows drive the fake's `approvedPlan`, `TestDecisionVerb_AmendsPlan` also gains rows calling `planApprovedReader` over a temp anchor: an absent overview answers false; an approved and an unapproved plan answer their flag; a plan that does not parse is an error wrapping `errPlanUnparsed`.
`TestAmendmentPathspec` rows: a present record, an absent untracked record, an absent tracked record (a deletion to stage), and a tracked-check error.
`TestDiscussionCommitPathspec_IncludesParentReviewDir` gains a row: a present amendment record joins the discussion pathspec.
`decisionFake`, `decisionFake.deps` and `newDecisionFake` gain the amendment fields and seams: `appendAmendment` writes through `loomshed.AppendAmendment` and the fake keeps the bytes it wrote; `committedAmendment` answers a preset HEAD copy, or, with a `statusCommitLanded` flag, the bytes the call wrote; `commitAmendment` returns a preset error; and `approvedPlan` returns a preset answer and error.
A row that needs HEAD to hold the entry's heading after a failed commit presets `committedCopy` to a record holding it, since the fake's commit returns its error before it updates the copy.
The existing decision tests leave `amendsPlan` false and keep their expectations.

**ImpactSummary:** confined to the `decision` verb and its fake; `discussionCommitPathspec` also serves the CommitDiscussion seam, where an absent, untracked record adds nothing, so Discussion-Bouncer's commit is unchanged.
