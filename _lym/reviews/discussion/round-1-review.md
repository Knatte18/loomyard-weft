---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md#D3
    summary: D3 keys each file's ruling on its diff hash alone, so D3's own way forward, a trailer-only commit then lym loom resume, reuses the stored uncaused ruling and the row stays Stuck; only a revert clears it.
    fix: In D3, key a stored ruling on the file's diff hash together with the Loosened-Check trailers of the review range (or state that an uncaused ruling is never reused and is re-ruled on every pass). Add an acceptance criterion under D3 that a commit adding only a Loosened-Check trailer for an uncaused check, followed by lym loom resume, re-spawns the judge, rules the check caused, and lets Drift-Verify finish Done.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md#D4
    summary: D4 stores the send-back findings file in _lym/reviews/code/, the Code-Bouncer run directory, which the new generation D4 starts archives by rename, carrying back only the settle and generation records; the review input path and the path D6's friction note names then point at a file that is gone.
    fix: In D4, store each send-back findings file at a path outside the Code-Review run directory that a new generation does not archive, such as the loom scratch directory beside the Parent-Align fixer report, and state that path in D4 and D6. Add a criterion that the re-entered Code-Bouncer's reviewers can read the send-back file named in their review input.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D4
    summary: D4's fix-here fallback checks only the target row's entry gate, but a darn run with Code-Review budget 0 turns the perch off through zero_rounds with no entry gate, so the verb sends back, Code-Bouncer approves without reading the findings, Describe is skipped, and PR-Review re-enters with the same findings until the reroute cap halts the run.
    fix: In D4's fix-here list and its Bound, add that the verb answers fix-here when the target perch would approve without review, such as a Code-Review budget of 0 that stamps zero_rounds on Code-Bouncer, decided from the built rows and settings, never the recipe name. Add the case to the D4 fix-here acceptance criterion.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md#D4
    summary: D4 says a PR-findings input starts a full generation whose review input names the send-back file and calls that mechanism existing, but today only an amendment keeps a review input, the rubric reads one only through the amendment note, a FullGeneration Bouncer on an unchanged subject resettles with no review, and the ReviewInput seam returns one path while D4 wants two.
    fix: In D4, state the new mechanism - a send-back file newer than the settle forces a new full generation even on an unchanged subject, the generation carries the review input, and the reviewer is told of it outside the amendment note, and say how the send-back file and the Parent-Align fixer report are both reported (or which one wins). Correct D4's Rationale so it no longer says the review input and full generation already do this. Align D6's rubric Review-input sentence with that channel.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D4
    summary: D4 says the re-entered PR-Review asks the parent again, but askParentReview returns early on the record's parent_review_asked flag, which the first ask sets and the re-entry's recordPullRequest never clears, so no second review request is sent.
    fix: In D4, state that the PR-Review row clears parent_review_asked in the pull-request record when it clears the send-back mark, and add an acceptance criterion that a PR-Review re-entered after a send-back, with pr_review_ask_parent on, posts a new review request to the parent.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D4
    summary: The D4 criterion says the re-entered PR-Review row opens a new review session, but ensureSession resumes the recorded session id, the conversation the verb told to end its turn, and D4 states no mechanism that changes this.
    fix: Decide in D4 whether the send-back clears the record's session id so a new session starts, or the recorded session is resumed with a fresh review request, state it in D4, and reword the D4 re-entry acceptance criterion to match.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D4
    summary: D4 relies on the PR-Review agent running the verb and transcribing the parent's findings into the file, so the agent can skip the verb, pick the count and severities, or take any number of sub-threshold sets in turn, all landing after Drift-Recheck; neither D4's Bound nor D2's 'D4 keeps PR-Review's fixes small' states what this path lets through or what limits it.
    fix: Add to D4's Bound what the fix-here path lets through when the agent skips the verb, under-counts, or receives several sets each under pr_review_small_max, and what bounds each (for example a count of fix-here findings across the PR-Review episode, or Go handing the parent's findings to the verb); or state each as an accepted risk with its limit. Make D2's Bound sentence about PR-Review's fixes match.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D2
    summary: A Loosened-Check trailer clears both the D1 finding and D3's uncaused ruling on its text alone, and any committing session, Code-Burler-Fix included, can write it with an invented figure; D2 and D3 do not state that the measured cause is attested, not verified, nor name the bound that it reaches the parent through the regression check section.
    fix: Add to D2's Bound that a trailer's measured cause is the committer's attestation and is not re-measured by D1 or D3, and that its bound is that each loosened check and its cause are listed in the regression check section D3 adds, so the pull request body and the parent's review see them; or forbid the fixer from writing the trailer and say so.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D4
    summary: D4's dirty-tree refusal is checked only when the verb runs, so the agent can still change the tree before the row's wake stops the session, and the reroute then hands Code-Bouncer a dirty worktree although D4 promises the wake meets a clean tree.
    fix: In D4, state that the PR-Review row's wake that acts on the send-back mark stops the session first and re-reads the tree, as it does for the continue flag, and keeps waiting on a dirty tree instead of rerouting.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:44
    summary: D2's rule that Go appends the trailer lines a role's handoff carries to a Go-written commit is testable behaviour with no acceptance criterion; D2's only criterion is the Review check of the shared fragment.
    fix: Add an acceptance criterion naming D2 that a commit Go writes for a role carries the Loosened-Check trailer lines that role's handoff gives.
  - id: F11
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:148
    summary: The Auto-mode assumption that a single operator change request is no findings set restates what D4's last paragraph already decides.
    fix: Delete the bullet under Auto-mode assumptions, or replace it with a pointer to D4 if the section must hold one; the support log's Question ledger already records the auto-pick.
---

The record covers the board entry's three ideas and the Scope's Out list matches the entry's own split, with domain rules left to each target's PATTERN.
Three defects stop the design from working as written: D3's trailer way forward cannot clear its own Stuck, D4's send-back file is archived before its reviewer reads it, and D4 sends back into a perch that is turned off.

### [BLOCKING] D3's trailer way forward never clears an uncaused Stuck

**Location:** `_lym/discussion/decision-record.md` D3, with D2's "with no change if need be".

**Issue:** Each file's ruling is "keyed by a hash of the file's diff, and reuse[d] while that diff is unchanged".
The way forward from an `uncaused` Stuck is a commit carrying the trailer, which D2 says may hold no change.
That commit leaves the file's diff unchanged, so on `lym loom resume` Drift-Verify reuses the stored `uncaused` ruling, spawns no judge and halts again (reuse path at `internal/driftengine/verify.go:246`).
The D3 criteria check only that the ways forward are named, not that following one works.
All three lenses that walked D3 reached this independently.

### [BLOCKING] D4 stores the send-back file where the new generation archives it

**Location:** `_lym/discussion/decision-record.md` D4 ("stores the file in the run's `_lym/reviews/code/` directory") and D6.

**Issue:** `_lym/reviews/code/` is the Code-Review perch's run directory (`RunRoot` is `LoomReviewsDir` in `internal/loomcli/wiring.go`, `run_subdir: code` on the rows).
A new generation renames it to an archive sibling and carries back only its records (`internal/shedadapters/archive.go`), so the input path given to the reviewers and the path the friction note names no longer exist.
The Parent-Align fixer report avoids this because it lives in the loom scratch directory.

### [BLOCKING] D4 sends back into a Code-Review perch that is turned off

**Location:** `_lym/discussion/decision-record.md` D4, the fix-here list and its Bound.

**Issue:** A Code-Review budget of 0 turns the perch off by stamping `zero_rounds` on Code-Bouncer, which settles approved at once and carries no entry gate (`internal/loomrecipe/doc.go`); Describe is skipped by `budget.Code-Review`.
D4's fallback checks only the target row's entry gate, so the verb sends back, the findings go unread, PR-Review gets them again, and the loop runs to the reroute cap.

### [MEDIUM] D4's full generation with a review input is new, not existing

**Location:** `_lym/discussion/decision-record.md` D4 and its Rationale; D6's rubric sentence.

**Issue:** In `internal/shedadapters/amendment.go`, only an amendment keeps a review input, and the rubric's `## Review input` reads one only "When the amendment note names a review input".
A Bouncer on an unchanged subject resettles with no review unless a newer input forces an amendment, and the send-back usually comes with no new commits, since the brief has the agent run the verb before fixing.
The `ReviewInput` seam returns one path, while D4 wants the send-back file and the fixer report.
The Rationale's "only the mark, the verb and the second input source are new" therefore misleads the plan writer.

### [MEDIUM] The parent is not asked again after a send-back

**Location:** `_lym/discussion/decision-record.md` D4, "asks the parent again".

**Issue:** `askParentReview` returns early on `ParentReviewAsked` (`internal/landingshed/prreview.go:345`), and `recordPullRequest` on a found record leaves the flag set.
D4 states the re-ask as existing behaviour, so no plan will add the reset.

### [MEDIUM] The re-entry session criterion contradicts the resume path

**Location:** `_lym/discussion/decision-record.md` D4 and its re-entry criterion.

**Issue:** `ensureSession` resumes the recorded `SessionID` when the strand is not live (`internal/landingshed/prsession.go`).
The criterion's "opens a new review session" has no decision behind it.

### [MEDIUM] Skipping or shaping the verb is an unbounded path

**Location:** `_lym/discussion/decision-record.md` D4 Bound and D2 Bound.

**Issue:** The verb is the agent's to run, and the findings file is the agent's own transcription of the parent's creel answer, which the row never reads (`prreview.go`'s `askParentReview` doc).
Skipping the verb, choosing the count or severities, and taking successive sub-threshold sets each let a large fix land after Drift-Recheck.
The rubric's attack-surface bullet asks the Decision to state what such a path lets through and what bounds it; D4 states neither, and D2's "D4 keeps PR-Review's fixes small" assumes the verb runs.

### [MEDIUM] The trailer's measured cause is unverified, and the record does not say so

**Location:** `_lym/discussion/decision-record.md` D2 Bound, with D1 and D3.

**Issue:** The focus asks what the trailer lets through and what bounds a cause nobody verifies.
D2's Bound speaks only of commits after Drift-Recheck.
The real bound exists, since D3 lists each loosened check and its cause in the regression check section the parent reads, but the record never names it as the bound, and nothing stops the fixer from clearing a D1 finding with an invented figure.

### [LOW] The dirty-tree refusal is checked only when the verb runs

**Location:** `_lym/discussion/decision-record.md` D4, the dirty-worktree refusal.

**Issue:** The session runs until the row's verdict ends it (`Call`'s `endSession`), so edits after the verb's answer reach Code-Bouncer as a dirty tree, despite "so the row's wake meets a clean tree".

### [LOW] D2's Go-appended trailer lines have no criterion

**Location:** `_lym/discussion/decision-record.md:44`.

**Issue:** The focus names this line, which is behaviour a test can check, yet no acceptance bullet covers it.

### [NIT] The Auto-mode assumption duplicates D4

**Location:** `_lym/discussion/decision-record.md:148`.

**Issue:** D4's last paragraph already rules that a single operator change request is no findings set.

## Rejected

- Tool-fact focus (review base as last merged parent tip, reroute edge with no bounce budget, reroute cap, Stuck verdict): every one is a claim about this repository's own code, confirmed in `contracts/stencils/loom/loom-rubric-code-review.md` and `internal/shedengine/reroute.go`, `doc.go` and `run.go`, so none is an unpinned tool fact.
- Open risk on the peer run `drift-judge-and-line-range`: a textual merge in `internal/driftengine` with no conflicting design; its fix (the judge snapshot check, line-range matching) does not touch D3's rulings, so it is a fair risk line and not a parent `decision`.
- No question only the parent can settle hides in the Decisions; every open point above is a choice the writer can make.
