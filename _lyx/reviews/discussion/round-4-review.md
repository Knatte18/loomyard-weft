---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 (the budget-0 cap paragraph, "measured as the tree diff from the HEAD at the row's start")
    summary: The Merge-In size and kind cap at a Code-Review budget of 0 measures from the HEAD at the row's start, which precedes the merge on a first start, so the parent's merged commits and the conflict resolution count as the fixer's change
    fix: >-
      In D4, replace "measured as the tree diff from the HEAD at the row's start, a resumed start included" with a baseline that is the HEAD after the merge and the conflict session have committed and before the fixer's first turn.
      State that Merge-In records this baseline in the attempt record in the run's scratch directory when the fixer first spawns, so a resumed Merge-In measures from the same HEAD, and that the cap judges only the fixer's own commits.
      Keep D10's pre-fix HEAD wording for Finalize, which already has this meaning.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D13 (orch role stencil rule) and D10 and D11 (reroute-cap halt)
    summary: D13 says a Finalize reroute-cap halt means a parent that keeps moving, but D10 also reroutes on a spent fixer cap and an over-cap fix, and the told reroute-cap way-forward clause names only a parent-branch settle and a Webster plan conflict
    fix: >-
      In D11, state that the reroute-cap way-forward clause (`loomRerouteCapWayForward` in `internal/loomcli/wiring.go`, the told `Shed.RerouteCapWayForward`) gains the Finalize causes that are not a parent settling, a fixer whose attempts are spent or whose change is over the cap, with the way forward to fix the failing tests on the task branch, then `lyx loom resume`, and that a test reaches the clause.
      In D13, reword the orch role stencil's rule so a reroute-cap halt at Finalize names its cause as a parent that keeps moving or a fixer that keeps failing to clear the gate within its caps, read from the halt's reason.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D13 ("such a run's deploy is bounded only by the landing gate, D10's size and kind caps ... and the operator's `deploy_branches` list")
    summary: D13's list of what bounds a budget-0 deploy omits the conflict session's resolution, which D10 step 3 says lands in the budget-0 darn recipe on the next Finalize pass's gate alone
    fix: >-
      In D13, add to the "bounded only by" sentence that a conflict the conflict session resolved, at a darn Finalize under the lock and at a loom Merge-In, is not a fixer change and no size or kind cap covers it, so at a perch budget of 0 it lands with the landing gate alone, as D10 step 3 states for the budget-0 darn recipe.
      Add the same line to Open risks.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D10 ("a record that no longer names this run means an owner pause or another machine's ownership released it")
    summary: D10 names only the owner-pause and owned-elsewhere arms as what can release a Finalize's lock record, but an operator's `lyx batten lock release` force-releases any record, and the halt's resume wording assumes the run is not yet this machine's
    fix: >-
      In D10, reword the sentence to "a record that no longer names this run means something released it, an owner pause, another machine's ownership or an operator's `lyx batten lock release`", and make the halt's way forward `lyx loom resume`, adding "once the run is this machine's" only for the owner and ownership causes.
      Say in the same place that the re-check stands behind every release the idle paths do not guard, so the plan writer does not add a guard per release path.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D4 ("the row keeps the attempts its entry spent in a record in the run's scratch directory, so no resume grants a fresh cap")
    summary: D4 does not say when the persisted attempt record resets, so a Merge-In entered again by a Finalize reroute may read the earlier entry's spent attempts as its own
    fix: >-
      In D4, state that the attempt record belongs to one row entry, that a resume continues the entry and reads the record, and that a re-entry of Merge-In or Finalize from another row, such as a Finalize reroute to Merge-In, starts at zero spent attempts, with the record cleared at the row's Done and at that re-entry.
      Add a case for the reset to the acceptance criteria on the Finalize reroute.
---

### [MEDIUM] Merge-In's budget-0 cap measures from a HEAD that precedes the merge

**Location:** `_lyx/discussion/decision-record.md` D4, the paragraph beginning "When the Code-Review perch's effective budget is 0".

**Issue:** D4 binds the `merge-in` fixer's change to D10's size and kind caps "measured as the tree diff from the HEAD at the row's start, a resumed start included".
At a first Merge-In, the row's start is before the merge, and the merge brings the parent's commits in as a merge commit (D2, through `MergeInAt`), so the tree diff from that HEAD holds every parent change and any conflict resolution.
Any merge that brings in more than `finalize_fix_max_lines` lines, or a changed test file of the plan's language, then trips the cap, and the run halts with the landing-stop prefix whatever the fixer changed.
At a resumed start the merge is already committed, so the same words measure only the fixer's commits, and a first start and a resume disagree.
D10 states the Finalize measure correctly, as the tree diff from the pre-fix HEAD.
The focus question on the budget-0 bound is answered in part: the record does state that D10's caps bind both fixers, but the Merge-In measure as written does not bind the fixer's own change.

### [MEDIUM] The Finalize reroute-cap halt is explained as a moving parent alone

**Location:** `_lyx/discussion/decision-record.md` D13 (orch stencil rule), D10 and D11; `internal/loomcli/wiring.go:1141`.

**Issue:** D13 rewrites the orch stencil so that "a reroute-cap halt at Finalize means a parent that keeps moving".
D10 gives Finalize four reroute causes: a loom conflict, a large merge, a spent fixer cap and a fixer change over the cap.
Two of these are not the parent.
The halt's way-forward clause is not the orch stencil's but the Shed's told `RerouteCapWayForward`, which `loomRerouteCapWayForward` sets to a text naming "a parent-branch reroute, once the parent branch has settled" and "a Webster plan conflict", and shed's `rerouteCapWayForward` appends it to the halt's reason.
The record leaves that clause untouched, so a Finalize halted by three exhausted fixers is told to wait for a parent that has settled, which `PATTERN-refusal-way-forward` and the record's own Constraint treat as a wrong way forward.
D11 hands exactly this halt to the orch, and the orch reads these two texts.

### [LOW] D13's bound on a budget-0 deploy omits the conflict session

**Location:** `_lyx/discussion/decision-record.md` D13, "With the PR-Gate guard removed, such a run's deploy is bounded only by ..."; D10 step 3.

**Issue:** D13 lists the landing gate, D10's caps over every fixer change and `deploy_branches` as the whole bound on a run whose perch budget is 0.
D10 step 3 says that in the budget-0 darn recipe the reroute after a large merge reaches the PR-Review stub, "so the operator's zero budget leaves the large merge to the next Finalize pass's gate alone", and step 2 has a darn Finalize resolve a conflict with the conflict session under the lock.
That resolution is LLM-written code that lands with no review and no cap, and a loom Merge-In's conflict session is in the same position at budget 0.
The rubric's attack-surface question asks the Decision to state what a removed guard can let through; D13 states a bound that is smaller than D10's own account of it.
This is not new to the tail, since a budget-0 loom run already skipped Merge-Review, but the Decision that removes PR-Gate should say so.

### [LOW] Release causes of a Finalize's lock record are narrower than the code's

**Location:** `_lyx/discussion/decision-record.md` D10, the paragraph on Finalize's re-check; `internal/battencli/lock.go` (`runLockRelease`, `ForceRelease`, `ReleaseAll`).

**Issue:** D10 guards the two idle paths, keeps the two batten arms releasing, and re-checks the record before landing, which is the right backstop.
It then says that a record no longer naming the run "means an owner pause or another machine's ownership released it".
`lyx batten lock release <branch>` and `--all` remove any record whatever its owner, and the store's own note says the released run "reacquires at its next lock-taking row", which Finalize no longer has.
The re-check catches that release too, so the behavior is right, but the stated cause and the resume wording ("once the run is this machine's") are wrong for it, and a plan writer reading the Decision could add a guard for a path the re-check already stands behind.

### [LOW] No reset rule for the persisted fixer attempt record

**Location:** `_lyx/discussion/decision-record.md` D4, the paragraph on a failed spawn and the attempt record.

**Issue:** D4 caps fixer turns per "row entry" and keeps them "in a record in the run's scratch directory, so no resume grants a fresh cap".
It does not say when that record ends.
A Finalize reroute to Merge-In is a new entry to the row, and D10 relies on the Merge-In that follows to run the fixer again over what Finalize left; if the record from the earlier entry stands, that Merge-In has no attempts left and halts at once on a failing gate.
Resume and re-entry need opposite treatment, and only the first is written.

## Rejected

- The focus's D11 question on the `binary_changed` sites: the record names the signal (`hand_back` on the stop envelope and `lyx shed status`, set by a producer-and-prefix check told through the Spec) and all three sites, and I found no other site that re-steps on `binary_changed` in the stencils (`grep` over `contracts/` and `internal/` finds the field only in the driver stencil, its guide, `shedverbs` and the Go driver's and batten's own re-step arms, which D11 covers). Not a defect.
- The focus's D2 question: the record names `mergeresolve.Resolve(ctx, source string)`, its `Fabric` field of type `MergeSurface` in `deps.go`, and `MergeInAt`, and Scope lists `internal/mergeresolve` for the interface change. How the landing package's unexported `resolver` interface and the loomcli wiring follow is call-site enumeration, which the rubric leaves to Plan-Write and the compiler. Not a defect.
- The focus's D10 step 4 language question: the record states where the language is read (a loom plan's `language:` with an absent key read as `go`, as `effectiveLanguage` in `internal/planparser/reviewclass.go` reads it; the union for darn; every file for `none` or a language the table lacks), and the code matches. Not a defect.
- The scope list not naming `internal/loomcli` or `internal/battencli` for the `hand_back` check's wiring: call-site enumeration, outside the rubric.
