---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/battencli/cancel_integration_test.go
    summary: >-
      No test runs cancel --remove through the production seams (RemovePair with Force, Snapshot, Remote and LeaveClaim, then DiscardTaskBranch), and stopPairStrands and liftSlugPauses have no test beyond their pure helpers.
    fix: >-
      Add one tier-1 row to cancel_integration_test.go over a hubforge hub with a pair and an unlanded task branch, run as cancel with --remove, asserting the pair is removed, the snapshot tag is taken, the task branch is gone locally and on origin under an archive tag, and the claim is cleared.
      Add a case to the same file or a tmux-tier test that stops a live strand through stopPairStrands, and a case for liftSlugPauses lifting one slug entry and keeping the scopes lifted before a failing lift.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/battencli/cancel.go:403
    summary: >-
      With remove and an unreachable origin, a slug whose only remnant is an origin task branch is refused as nothing left to cancel with a board list way forward, which misleads since origin was not read.
    fix: >-
      In refuseNothingLeft, when remove is set, endOnly is true and no local part is left, return a refusal that says origin was unreachable so the task branch on origin could not be read, and names rerunning "lyx batten cancel <slug> --remove" once origin answers, instead of the nothing-left text.
      Add a TestCancelRun row for it.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/loom-status-spec.md:43
    summary: >-
      The status spec lists the blocked halt reasons and documents the bounce budget exhausted prefix, but not the new precondition unmet prefix that Worktree-Create now writes.
    fix: >-
      In contracts/specs/loom-status-spec.md next to the bounce budget exhausted entry at line 43, add the precondition unmet prefix, stating that a producer sets it when a precondition the run cannot settle on its own is unmet, that the reason continues with what is unmet and a trailing way forward clause, and that the driver hands such a halt back and never resumes it.
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-pattern-fit
    location: internal/reedengine/parse.go
    summary: >-
      Commit 0e0c9df2 changes the reedengine pane-tail separator to a tab, which no plan card names; every other commit in the range is a numbered card.
    fix: >-
      Remove the reedengine changes of commit 0e0c9df2 from this branch and land them as their own change, or have the orch confirm in the run record that the commit is intended here.
  - id: F5
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/battencli/startguard.go:143
    summary: >-
      refuseAtStart reports every ReadJSONNoMkdir error, a lock failure included, as a decode failure and drops the cause chain.
    fix: >-
      In refuseAtStart, wrap the error with %w and word it as reading the status file, for example fmt.Errorf("battencli: read status file %s: %w", statusPath, err).
---

### [LOW] Production seams of the cancel's removal are untested

**Location:** internal/battencli/cancel_integration_test.go
**Issue:** The plan's two integration tests cover the no-pair acceptance and the judge-only reconcile. The forced origin-mode removal followed by the task-branch discard, the strand stop and the pause lift run only through fakes in `TestCancelRun`, so the focus question on whether the production seams match what `cancelRun` expects has no test behind it. Each half is pinned alone (`TestDiscardTaskBranch`, `TestTeardown_CallSequence`).

### [LOW] Misleading nothing-left refusal when origin is unreachable

**Location:** internal/battencli/cancel.go:403
**Issue:** The origin read is skipped in end-only mode, so a slug with only an origin task branch reads as nothing left, and the way forward points at the board list.

### [LOW] Status spec omits the precondition halt reason

**Location:** contracts/specs/loom-status-spec.md:43
**Issue:** The spec is the one place that lists the blocked halt reasons, and the diff does not touch it. The repository's docs-in-the-same-commit rule covers a new observable halt reason.

### [LOW] Unplanned reedengine commit in the range

**Location:** internal/reedengine/parse.go
**Issue:** Commit 0e0c9df2 is outside the plan's card index. It is a tmux 3.4 compatibility fix and unrelated to the cancel verb.

### [NIT] Status read error worded as a decode failure

**Location:** internal/battencli/startguard.go:143
**Issue:** A lock or read error is reported as "decode status file" with the cause stringified.

## Rejected

- Windows first-list refusal of every live registered process (code-correctness): card 18 step 4 specifies it, and the plan is the measuring stick, not the subject.
- Fresh marker left after a `LiveRecords` probe error on Windows (code-correctness): Windows-only, unspecified by the plan, and a rerun of the cancel finishes it.
- End-only with `--remove` passing `Keep: false` (code-correctness): card 18 step 5 specifies `Keep: !remove`, and a rerun recovers it.
- `lyx batten run` does not map `ErrLockedCheckRefused` to a kind (code-affected-set): the refusal still reaches the envelope with its message, and the driver steps through `step`, which is mapped.
- `TestJudgeCancelMarker` rows repeat `TestCancelMarker` (code-tests): card 15's lens review asked for the pure helper's rows, and the race row is unique.

All other checks held: the declared `PATTERN-pair-teardown` text landed in `pattern/PATTERN-pair-teardown.md`, `cancelRun` follows D2 to D9 with the run-lock wait after the owned processes and strands are ended, the board claim is cleared by the cancel (`LeaveClaim`), and no commit carries a `Pattern-Departure:` trailer.
