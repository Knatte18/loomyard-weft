---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/03-move-begun-card-hashes.md (Bound)
    summary: Card 3's bound claims a rewrite never runs while an unadopted edit is on disk, but every restamp site has a window between its foreign-edit check and its restamp in which an edit lands and is adopted, and card 3 extends that adoption to begun CardHashes
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/09-integration-fix-attempt.md (Docs, Tests)
    summary: The new refusal-spec row has no reaching test asserting the stuck reason names the way forward, and the row's way forward stops at commit without saying how the run continues
  - id: F3
    severity: LOW
    location: _lyx/plan/09-integration-fix-attempt.md (steps 1-2)
    summary: Step 1 spends the attempt before the nil-FixStarter wiring guard and before rendering, so a wiring or render error leaves a spent attempt with no Result and no attempt ever made
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/07-fix-commit-check.md (Intent)
    summary: In the hub geometry _lyx is a directory link out of the task repository, so no commit can touch the plan directory and the commit check passes an on-disk plan write; the card states no bound for that case
  - id: F5
    severity: MEDIUM
    location: _lyx/plan/05-integration-fix-prompt.md (Intent)
    summary: The prompt forbids touching anything under _lyx yet tells the strand to write its report, and its friction note, under _lyx
  - id: F6
    severity: MEDIUM
    location: _lyx/plan/09-integration-fix-attempt.md (steps 1-6)
    summary: A run killed while the fix strand is in flight resumes with the strand's commits on the branch, and when the resumed triage reads OK or flaky they reach outcome done without checkFixCommits and without a summary.md record
  - id: F7
    severity: MEDIUM
    location: _lyx/plan/07-fix-commit-check.md (Intent)
    summary: Nothing checks the worktree is clean after the strand, so the post-fix verify can pass on an uncommitted change the branch does not carry
  - id: F8
    severity: LOW
    location: _lyx/plan/01-dispatch-forthcoming-targets.md (Intent)
    summary: The current plan the forthcoming cards are selected from reads as resolvePass's current variable, which holds pending cards only and so never contains a forthcoming card
---

### [MEDIUM] Card 3's bound overstates what the foreign-edit check covers

**Location:** `_lyx/plan/03-move-begun-card-hashes.md`, the `Bound:` paragraph.

**Issue:** Checked against each site the card lists:

- begin-batch (`beginbatch.go:280`→`303`): `PlanEditError`, then `ValidateDispatch` (quarry), then the restamp.
- `Run` (`runlevel.go`): the fingerprint is computed before `AcquireStateMutation`, compared after `LoadState` and the entry-time reclaim, and `restampAndSaveFingerprint` follows `ValidateDispatch`.
- record-batch (`recordbatch.go:273`→`601`/`644`): `PlanEditError` and `batchCardEditError`, then `rerunCardVerifies` (when a policy finding is present), the done-checks and `planglyph.Delta`, then the two restamps.
- recovery (`recoverbatch.go:418`→`postBatchChecks`): the same shape.
- validate (`validate.go:242`→`254`): restamps only when `editErr == nil`.

No site restamps after a check that failed: begin-batch and run refuse before any rewrite, record-batch and recovery refuse before `postBatchChecks`, and validate skips the restamp.
The Shared Decision's "persisted even when the call fails" therefore applies only to calls whose check passed.
But every site has a window between its check and its restamp, minutes long in record-batch when card verifies re-run.
An edit landing in that window is adopted along with the rewrite.
Today a begun card stays protected through that window: its `CardHashes` stay put, so `batchCardEditError` still refuses it later.
Under card 3, `moveBegunCardHashes` sees recorded hash == pre-rewrite `PlanFileHashes` entry and moves it to the edited bytes, so the edit is adopted for good.
The bound's "a rewrite never runs while an unadopted edit is on disk" is not true, and the card never states the residual.

**Fix:** Restate the bound: the check runs before the rewrite, not atomically with the restamp, so an edit landing between the two is adopted with the rewrite, begun cards included.
Name the window (the check-to-restamp span of one call: quarry resolution at begin-batch and run, card-verify re-runs and done-checks at record-batch and recovery) and its limit (the edit must land during that one call; any edit before it is refused).

### [MEDIUM] Card 9's refusal row lacks a way-forward test and a resume step

**Location:** `_lyx/plan/09-integration-fix-attempt.md`, the Docs paragraph and the Tests list.

**Issue:** The Refusal Way-Forward Invariant requires a new refusal to land with its row and its reaching test in the same commit, and each refusal to name its way forward in its message.
Today's integration escalation has no row (`refusal-spec.md` has none for it), and `triageStuckReason` names no way forward.
Card 9 adds the row and extends the stuck reason, but its tests assert only `-1` record, outcome stuck and the summary record.
None asserts that the returned stuck reason carries the way forward the row states: the pre-fix head, the fix commits, `git reset --hard <pre-fix head>`, fixing on the task branch, running the verify and committing.
The row's way forward also ends at "commit" without saying how the run continues, unlike every other webster row that ends a run (`re-run lyx webster run` / re-step the Webster row).

**Fix:** Add to the way forward "then re-run `lyx webster run` (re-step the Webster row)", in both the row and the stuck reason.
Add a test asserting the stuck reason of an unfixed attempt and of a spent attempt names the pre-fix head, each fix commit, `git reset --hard`, and the re-run.

### [LOW] Card 9 spends the attempt before its wiring guard and render

**Location:** `_lyx/plan/09-integration-fix-attempt.md`, steps 1–2 and the nil-`FixStarter` paragraph.

**Issue:** Step 1 records `IntegrationFix` and saves; the nil-`FixStarter` guard fires "when a regression reaches step 2", after that save.
The guard returns a wiring error, so phase 3 never runs and `IntegrationFix.Result` stays empty; a resumed run then reads the attempt as `spent` though no strand ever started.
A start error is named a failed attempt, so it reaches phase 3 and sets `Result`; a render error at step 2 is not classified at all.
D5 records the attempt "when spawned"; recording it just before the spawn is the crash-safe reading, but a pure wiring fault should not spend it.

**Fix:** Check `FixStarter` before step 1, so a wiring error spends nothing.
Name a render error, like a start error, a failed attempt (result `failed`, detail naming the error) that reaches phase 3 and sets `Result`.

### [MEDIUM] Card 7 states no bound for a plan directory outside the repository

**Location:** `_lyx/plan/07-fix-commit-check.md`, the Intent's path rule.

**Issue:** The card checks `planDir` "taken relative to `worktree` when it lies inside it" and states nothing about the other case.
That case is the hub's normal geometry: in this worktree `_lyx` is a directory link to the weft worktree and is not tracked (`git ls-files _lyx` is empty), so `PlanDir` (`<anchor>/_lyx/plan`) is outside the task repository.
No commit can change a file there, so the plan and `_lyx` checks pass vacuously, and only a commit adding or retargeting the `_lyx` entry itself is visible.
A strand that edits plan files on disk without committing passes the check; only the prompt forbids it.
The `_lyx` path check bounds commits, not that write.
Webster already holds what Go needs to catch a plan write: the recorded plan fingerprint.

**Fix:** In card 9, capture the plan fingerprint at step 1 and fail the attempt as `refused` when it differs after the strand (step 5).
In card 7, state the bound: commit paths are checked against `planDir` and against the `_lyx` entry itself or anything under it; when either lies outside the repository no commit can reach it, the plan is guarded by card 9's fingerprint compare instead, and other on-disk writes under `_lyx` are bounded by the prompt alone.

### [MEDIUM] Card 5's prompt forbids writing where its report and friction note go

**Location:** `_lyx/plan/05-integration-fix-prompt.md`, the bullets under the stencil.

**Issue:** The stencil tells the strand to "never touch `{{.plan_dir}}` or anything under `_lyx`", then to write `{{.report_path}}` as its last action.
Card 9 places that report under the reports directory, `websterengine.ReportsDir`, which is `<anchor>/_lyx/webster/reports`.
The friction directive's note path also sits under `_lyx`.
A cold strand reading the two rules either refuses to write the report, which turns a real fix into a failed attempt, or learns the `_lyx` rule is soft.

**Fix:** Word the rule as: never commit anything under `_lyx`, never write under `{{.plan_dir}}`, and write nothing else under `_lyx` but `{{.report_path}}` and the friction note the directive names.
Have the render test assert the report path is the stated exception.

### [MEDIUM] Card 9 lets an interrupted attempt's commits through on resume

**Location:** `_lyx/plan/09-integration-fix-attempt.md`, steps 1–6 and the Bound.

**Issue:** If the run process dies while the strand is in flight (killed, crashed, the host restarted), state.json holds `IntegrationFix` with an empty `Result` and the strand's commits sit on the branch.
Card 6 reclaims the strand at the next `Run` entry, and the fresh Master re-drives to the integration stage.
When that verify passes (report `OK`) or the triage reads flaky, the stage returns without escalating, so the run ends done on commits `checkFixCommits` never saw and summary.md never names.
Only a regression reaches step 1's `spent` branch.
D5's bounds (Go accepts the commits only after the check; summary.md names them for review) are bypassed on that path.

**Fix:** In the stage, after the report loads and before the `OK` early return, treat a loaded `IntegrationFix` with an empty `Result` as an interrupted attempt.
End it as `failed` with a detail naming the interruption, record `Fix` with the commits from `PreFixHead` to HEAD, append `AppendIntegrationFix`'s section, set `Result`, and return a stuck reason with the way forward.
Add a test: a state with an in-flight record and an `OK` integration report ends stuck, and summary.md names the commits.

### [MEDIUM] Card 7 accepts a dirty worktree, so the post-fix verify can judge uncommitted work

**Location:** `_lyx/plan/07-fix-commit-check.md`, the conditions list; card 9 step 6.

**Issue:** D5 has the strand fix "at HEAD, commits" and lets the post-fix triage decide success.
Card 9 step 6 runs the verify in the worktree, which includes any uncommitted change the strand left.
A strand that edits without committing (or commits only part) clears the triage while the branch's commits do not carry the fix, and the run ends done on a branch that still regresses.
Card 7 checks commits only, and record-batch's own `dirty` call only warns.

**Fix:** Add a condition to `checkFixCommits`: the worktree has no uncommitted or untracked change (the existing `dirty` helper in `gitwrap.go`); otherwise the attempt counts as failed with an error naming the dirty state.
Add the matching test case.

### [LOW] Card 1 names the wrong plan to select forthcoming cards from

**Location:** `_lyx/plan/01-dispatch-forthcoming-targets.md`, the `resolvePass` sentence.

**Issue:** "selects those cards by ID from the current plan" reads as `resolvePass`'s `current` variable, which is `pendingCardsByID(..., done)` and holds no completed card.
Forthcoming cards are a subset of `completed`, so selecting from `current` returns nothing and the fix is a silent no-op.

**Fix:** Say: select them from the reloaded plan when canonicalization rewrote it, and from `plan` otherwise; never from `current`, which holds pending cards only.

## Focus verdicts not raised as findings

- Card 8's Describe stencil edit is licensed.
  Webster-Review rows judge the committed diff against the plan and never read summary.md.
  The Describe stencil is the existing route by which summary.md's integration sections (`## Integration suite failed`, `## Integration suite triage`) reach a reviewer.
  Carrying the new section the same way is how D5's "summary.md names the fix commits ... for the review rows below to judge" reaches anyone.
- Verify coverage holds: every targeted package runs plain and `-tags integration`, the smoke tag is compiled only, and `go test ./...` covers `internal/landingshed`'s Describe tests.
