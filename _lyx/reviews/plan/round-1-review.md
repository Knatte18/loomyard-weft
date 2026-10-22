---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/03-darn-failure-classifier.md:152
    summary: notOwnedRoute's way forward makes the Merge-In goto conditional on the failure also failing on the parent, so a flake that passes on the parent and that the task did not cause matches neither clause, while D2 makes only the parent fix conditional
    fix: In card 3's notOwnedRoute way-forward text, make the parent fix the conditional part and the Merge-In goto unconditional, following D2's order, e.g. `fix the failure on the parent branch if it fails there too, then run "lyx loom goto --to Merge-In", then "lyx loom resume", in the task worktree, which merges a moved parent and re-runs the failing gate; or, if the task did cause it, run "lyx loom goto --to Darn-Write", then "lyx loom resume", in the task worktree, where a moved parent is merged in first`. Also update the not-owned row of TestDarnFailureRoute that asserts the way forward.
    origin: lens:way-forward
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/09-darn-write-reroute-edge.md
    summary: docs/overview.md:547 says Publish, PR-Gate and Finalize reroute to Merge-In when the parent moved, and no card updates it although card 9 gives Darn-Write a Merge-In reroute edge
    fix: Add `docs/overview.md` to card 9's Edit list, and add an Intent sentence saying the landing-tail paragraph's reroute line (docs/overview.md:547) names Darn-Write's reroute to Merge-In, taken when this run's failure record is present, the branch and worktree are unchanged since it, and the parent moved. Checked with validate-plan --overlay; it adds no finding.
    origin: lens:affected-set
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/10-darn-write-entry-reroute.md:425
    summary: the no-parent-tip branch of rerouteReason and its TestDarnWrite_Call row pin a state the real ParentTips seam cannot produce, since ReadParentTips errors on a missing local branch and mergeableTips always keeps the local tip
    fix: In card 10, drop the writer-runs row "no parent tip" from TestDarnWrite_Call, and in the rerouteReason paragraph replace "no tip counting as not moved" with a note that the seam always returns the local tip, so a failed tip read is the only not-moved case short of every tip being an ancestor.
    origin: lens:guard-reach
---

### [LOW] The not-owned way forward ties the Merge-In goto to the parent failing too

**Location:** `_lyx/plan/03-darn-failure-classifier.md:152`, `notOwnedRoute`.

**Issue:** The planned text reads "if the failure also fails on the parent branch, fix it there, then run goto --to Merge-In …; if the task did cause it, run goto --to Darn-Write …".
Grammatically the Merge-In goto sits inside the first condition.
A flaky failure that passes on the parent and that the task did not cause matches neither clause unless its package drives the `lyx` binary.
That is the case D2 keeps the Merge-In re-run for.
D2 orders it as "fix the failure on the parent branch if it fails there too, then run `lyx loom goto --to Merge-In` and `lyx loom resume`", so only the fix is conditional.

### [LOW] `docs/overview.md` keeps listing three reroute rows

**Location:** `_lyx/plan/09-darn-write-reroute-edge.md`; the stale line is `docs/overview.md:547`.

**Issue:** The overview's landing-tail paragraph says "`Publish`, `PR-Gate` and `Finalize` reroute to `Merge-In` when the parent moved since."
Card 9 adds Darn-Write's `on_reroute: Merge-In` edge and updates the twin header line in `contracts/recipes/darn-recipe.yaml:33`, but no card edits the overview.
CLAUDE.md's "Docs land in the same commit" asks for `docs/overview.md` when the execution stack changes.
The decision record's doc list omits the overview, but the edit is follow-through on the edge D3 licenses, not new scope.

### [LOW] Card 10 tests a parent-tip state that cannot occur

**Location:** `_lyx/plan/10-darn-write-entry-reroute.md:425` and the "no parent tip" row at line 449.

**Issue:** `fabricengine.ReadParentTips` returns an error for a missing local branch (`internal/fabricengine/parenttips.go:42`), so `Local` is set on every success.
Card 10's `mergeableTips` always keeps the local tip, and a failed read already takes the separate failed-tip-read branch.
The empty-tips case is reachable only through the fixture.
A row that pins it tests a misuse the wiring rules out.

## Rejected

- guard-reach: card 3's opaque-identity branch and its row, which plants a record test beside an opaque-only log, guard a state production cannot produce. Rejected: D1 names an opaque-only log as its own owned condition, and an acceptance criterion requires it. The row exists so that the branch is pinned past the empty-tests check, which the classifier reads from a told seam and not from the log, so the redundancy holds only as long as the wiring does.

## Holistic notes

D1 to D6, each Constraint and each Acceptance criterion are carried by some card.
D3's gate-exhausted way forward is carried by card 10.
The package docs and the recipe header are carried by cards 1, 2, 4, 7, 9 and 10.
No card carries a `**Verify:**` line, card order holds against cards 5 to 8, and the re-sign arrows and ImpactSummaries on cards 1, 2, 3, 7 and 10 match their Intents.
Card 7's coverage of the gated producers holds: `gateFailedReason` has only the two production callers card 7 edits, and `WebsterProducer`'s Stuck reason comes from `verifyGateStuckReason` (`internal/websterengine/verifygate.go:450`), which names no findings path.
`Env.GateFindingsFile` is one geometry seam shared by every gated row, not a per-row value, so `PATTERN-recipe-setting-lookup` is not broken.
