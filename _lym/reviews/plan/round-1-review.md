---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/03-drift-verify-loosened-checks.md#Edit
    summary: Card 3 makes driftVerifyEntry require Drift.ChangedFiles and Drift.CommitMessages, but the shedrecipe test kit driftTestEnv fills neither, so TestDriftEntries' VerifyBuilds row fails at card 3's own gate; the card lists neither.
    fix: In _lym/plan/03-drift-verify-loosened-checks.md, add `internal/shedrecipe#driftTestEnv` and `internal/shedrecipe#TestDriftEntries` under **Edit:**. In the Intent, say that driftTestEnv fills Drift.ChangedFiles and Drift.CommitMessages with empty answers, and that TestDriftEntries gains a nil-seam row for each of them, beside its existing VerifyNilCompare-style rows.
  - id: F2
    severity: BLOCKING
    class: scope
    origin: lens:affected-set
    location: _lym/plan/10-recipes-pr-review-reroute.md#Edit
    summary: Card 10 gives tapestry's PR-Review on_reroute Code-Bouncer, but internal/loomrecipe#TestRouting_ProgressAt pins PR-Review's reroute as empty, and no card edits it, so card 10's own gate fails.
    fix: In _lym/plan/10-recipes-pr-review-reroute.md, add `internal/loomrecipe#TestRouting_ProgressAt` under **Edit:**, and say in the Intent that its tapestry-tail subtest's PR-Review row now wants reroute loomshed.NameCodeBouncer.
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/07-bouncer-review-inputs.md#Intent
    summary: Card 7 has burlerReviewEntry fill the review_input note from ReviewInputNote when the row is built, but the detached lym loom run builds the shed once, so after PR-Review reroutes in that process Code-Burler-Review's rubric still reads none; no test covers the reviewers' rubric.
    fix: "In _lym/plan/07-bouncer-review-inputs.md, state how Code-Burler-Review's reviewers get the review-input note of the generation the Bouncer starts in the same run process. Either have the Code-Bouncer seed name each input of the generation record in the round's focus file, which the reviewers read each round, or render the review_input note when the round's prompt is filled instead of in burlerReviewEntry. Add a TestBouncer_Amendment row, or a shedrecipe row, that builds the review row before the generation record names the send-back file and asserts that the round's reviewers are told its path."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-loom-pr-findings-verb.md#Intent
    summary: Neither card 8 nor card 12 says where the PR-Review agent writes the findings file, and verifytree.DirtyPaths counts untracked files, so a file in the task worktree trips the verb's own dirty-worktree refusal and the row's dirty-tree Stuck, whose commit-or-discard way forward commits the findings or deletes the input.
    fix: In _lym/plan/12-result-check-wording.md, have the PR-Review brief tell the agent to write the findings file under the task worktree's git-ignored .scratch/ directory. In _lym/plan/08-loom-pr-findings-verb.md, say the same in the verb's help text, and add a TestPRFindingsVerb row in which a findings file under an ignored path passes the dirty check and sends back.
  - id: F5
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/07-bouncer-review-inputs.md#Edit
    summary: Card 7 replaces Env.ReviewInput with ReviewInputs, a list with a full-generation flag, but contracts/specs/shed-recipe-spec.md:110 still says the Bouncer is told Env.ReviewInput, and no card lists the spec.
    fix: In _lym/plan/07-bouncer-review-inputs.md, add `//contracts/specs/shed-recipe-spec.md` under **Edit:**, and say in the Intent that its Bouncer passage names Env.ReviewInputs, every input written after the settle, and the full generation a full input forces.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/10-recipes-pr-review-reroute.md#Intent
    summary: After cards 9 and 10, PR-Review returns a Stuck with Reroute and no Halt, so 'Stuck is always a Halt' becomes false in internal/loomrecipe/doc.go:25 and in the PR-Review row comments of all three recipes; card 10's Intent rewords only the no-edge-out header comments, and no card lists the loomrecipe doc.go.
    fix: In _lym/plan/10-recipes-pr-review-reroute.md, add `//internal/loomrecipe/doc.go` under **Edit:**. Widen the Intent so it rewords internal/loomrecipe/doc.go:25 and the PR-Review row comments (tapestry-recipe.yaml:521-522, sampler-recipe.yaml:239-240, darn-recipe.yaml:30 and its PR-Review row comment) to say that PR-Review's Stuck is a Halt, except a marked send-back's reroute to Code-Bouncer.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/09-pr-review-send-back-row.md#Edit
    summary: The PRReview type doc (internal/landingshed/prreview.go:29-45) lists how the row's wait ends, and card 9 adds a send-back reroute ending, which falsifies it; the card does not list internal/landingshed#PRReview.
    fix: In _lym/plan/09-pr-review-send-back-row.md, add `internal/landingshed#PRReview` under **Edit:**, and say in the Intent that its doc comment names the send-back reroute to Code-Bouncer among the ways the row ends.
  - id: F8
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/03-drift-verify-loosened-checks.md#Intent
    summary: Card 3 adds an escape hatch, since a Loosened-Check trailer turns a loosened check caused and lets Drift pass, but its Intent never says the cause is the committer's attestation, which nothing re-measures, nor names its bound, the verdict listing of every loosened check with its cause (D2's Bound).
    fix: In _lym/plan/03-drift-verify-loosened-checks.md, add one line to the halt or verdict passage of the Intent. It says that a trailer's measured cause is the committer's attestation, which neither the judge nor the code re-measures, and that its bound is the verdict's Loosened checks lines, which carry each check with its ruling and cause into the pull request body and PR-Review's drift_verdict.
---

### [BLOCKING] Card 3 leaves shedrecipe's drift test kit without the two seams it now requires

**Location:** _lym/plan/03-drift-verify-loosened-checks.md#Edit
**Issue:** Card 3 adds `Drift.ChangedFiles` and `Drift.CommitMessages` to the seams `driftVerifyEntry` requires (`internal/shedrecipe/entries_drift.go:157-190`).
`internal/shedrecipe/entries_drift_test.go:20-60` builds `driftTestEnv` by hand and fills neither seam.
`TestDriftEntries` (line 62) has a `VerifyBuilds` row at line 102 that expects a built `DriftVerify`, so it fails once card 3 lands.
The card lists `envkit`'s `driftDeps` and `nilLegal` but not this kit. The lenses affected-set and repo-invariants both found it.

### [BLOCKING] Card 10 breaks the routing pin of TestRouting_ProgressAt

**Location:** _lym/plan/10-recipes-pr-review-reroute.md#Edit
**Issue:** `internal/loomrecipe/progress_test.go:60-82` asserts the tapestry tail's routing row by row.
It wants `{loomshed.NamePRReview, loomshed.NameFinalize, "", ""}`, so PR-Review's `OnReroute` must be empty.
Card 10 sets it to Code-Bouncer and lists only the three recipe files plus a new test, so the card fails its own gate.

### [MEDIUM] Code-Burler-Review's review-input note is fixed when the shed is built

**Location:** _lym/plan/07-bouncer-review-inputs.md#Intent
**Issue:** Card 7 says `burlerReviewEntry` sets `RubricNotes.ReviewInput` from `ReviewInputNote`, beside the amendment note.
`burlerReviewEntry` builds its notes and calls `ReadRubric` in the constructor (`internal/shedrecipe/entries_burler.go:125`, then `burlerRoundProfile` at line 315 fills `Profile.Rubric` once).
`lym loom run` calls `BuildShed` once and then runs `shed.Run` (`internal/shedverbs/run.go:42-47`), and the run is a detached `lym loom run` process.
So when PR-Review reroutes to Code-Bouncer inside that process, the Bouncer writes a generation record naming the send-back file, and its own seed and judge read it through `rubricNotes` at call time.
But Code-Burler-Review's rubric still holds the note rendered when the process started, `none`.
D4's criterion asks that the re-entered perch's reviewers can read the send-back file, and D6 asks that the rubric's `## Review input` section reads it.
Card 7's tests check only the Bouncer's generation record and `ReviewInputNote`, never the review row's rubric.

### [MEDIUM] The findings file has no stated location, so the verb's own input trips its dirty-tree refusal

**Location:** _lym/plan/08-loom-pr-findings-verb.md#Intent
**Issue:** `verifytree.DirtyPaths` reports every `git status --porcelain` path, untracked files included, and leaves out only ignored files (`internal/verifytree/verify.go:274-280`).
Card 8 wires the verb's dirty seam with it, and card 9's `sendBack` halts on a dirty tree.
Card 12's brief text tells the agent only to write the findings "first in burler's review format", with no location.
A file written in the task worktree therefore makes step 1 refuse the verb on its own input.
Both ways forward then offer "commit or discard": committing puts the findings file on the pull-request branch, and discarding deletes the input.
The worktree's `.scratch/` is git-ignored (`.gitignore:23`), so it is a location that passes the check.

### [MEDIUM] shed-recipe-spec still names Env.ReviewInput

**Location:** _lym/plan/07-bouncer-review-inputs.md#Edit
**Issue:** `contracts/specs/shed-recipe-spec.md:110` reads "Every kind also tells the Bouncer `Env.ReviewInput`, which stays optional."
Card 7 replaces that seam with `ReviewInputs`, which returns every input written after the settle and flags the inputs that force a full generation.
No card lists the spec, so the spec contradicts the code once card 7 lands.

### [LOW] "PR-Review's Stuck is always a Halt" stays in docs and recipe comments

**Location:** _lym/plan/10-recipes-pr-review-reroute.md#Intent
**Issue:** `internal/loomrecipe/doc.go:25` says "PR-Review's Stuck is always a Halt, so it carries no on_stuck".
The PR-Review row comments say the same: tapestry-recipe.yaml:522, sampler-recipe.yaml:240, and darn-recipe.yaml:30 with its row comment.
Card 9's send-back returns a Stuck with `Reroute` and no `Halt`.
Card 10's Intent rewords only the header comments that say PR-Review has no edge out.
`doc-go-unlisted` does not fire, because card 10 changes no non-test Go in loomrecipe.
The lenses affected-set and repo-invariants each found a part of it.

### [LOW] The PRReview type doc lists the row's endings without the send-back

**Location:** _lym/plan/09-pr-review-send-back-row.md#Edit
**Issue:** `internal/landingshed/prreview.go:29-45` documents the row's endings: "The wait ends when the pull request is merged or approved, or closed or continued by the operator."
Card 9 adds a reroute of a marked send-back, but it edits `Call`, `wake` and the package `doc.go`, never the type's doc comment.

### [LOW] Card 3 does not bound the trailer escape hatch

**Location:** _lym/plan/03-drift-verify-loosened-checks.md#Intent
**Issue:** With card 3, a `Loosened-Check:` trailer turns an uncaused halt into a `caused` ruling and lets Drift-Verify pass.
D2's Bound says the cause is the committer's attestation, which nothing re-measures, and that the verdict listing is its bound.
Card 3's Intent renders that listing but never states either point.
The rubric's attack-surface check asks the introducing card to state what the hatch lets through and what bounds it.

## Rejected

- `sendBackEdge` reading a target `MaxBounces` of 0 as a Code-Review budget of 0: this is sound. `ProducerDef.MaxBounces` documents 0 as "inherits", but `loomrecipe.New`'s `applyReviewBudget` sets every review-segment row to its resolved budget, and `stampZeroRounds` stamps `zero_rounds` on exactly that 0 (`internal/loomrecipe/loomrecipe.go:195-219`).
- The Shared Decision "no Go-written role commit today": the search claim holds. Go's code-side commits are the merge commits from `fabricengine.Merge` and `MergeContinue` and records commits; every role's code commit is the session's own (`internal/gitrepo/gitrepo.go`, `merge.go:129`). So D2's Go-append clause has no site, and card 11 carries D2 consistently.
- Card order: the order holds across every pair the focus names. Card 6 precedes 11, cards 4, 5 and 7 precede 8, card 5 precedes 9, card 2 precedes 3, and card 9 precedes 10. Before card 10, no mark can be written, since `sendBackEdge` answers fix-here on a row with no `OnReroute`.
- Test economy in cards 6, 7, 11 and 12: no test pins static prose. `TestStencils_TopLevelMarkers` and `TestLoomRubrics` pin marker sets, which PATTERN-test-speed allows, and card 12 plans no test.
- Refusal reach in cards 8 and 9: every refusal and guard named in the focus has a named reaching input and a test row. The guard-reach lens traced each one; the one gap it found is F4.
- Fidelity: D1 through D7 and every Constraint are carried. D1, D4's brief and D7 sit on card 12. D2 sits on card 11. D3 sits on cards 1 to 3. D4 sits on cards 5 and 7 to 10. D5 sits on card 4 and D6 on cards 7 and 8. No card writes the task's own board entry or files a task, no card leaves a fact to be found at run time, and no card carries a Verify line.
