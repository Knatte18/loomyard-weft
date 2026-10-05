---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: contracts/stencils/orch/orch-template-role.md, sections The run loop, After landing, PR-Gate
    summary: The orch role stencil prescribes fabric add before batten run and fabric remove after landing, though the batten recipe creates and tears down the pair itself, and says Finalize squash-merges while it pushes to main and closes the PR
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/loomcli/driverstencil_test.go, internal/shuttleengine/claudeengine skill-timeout test, internal/shuttleengine/session_test.go
    summary: New tests break PATTERN-test-economy - one test per stencil phrase, a test of a constant getter, and delegation tests asserting calls into own code
---

### [BLOCKING] The orch role stencil misstates the run loop

**Location:** `contracts/stencils/orch/orch-template-role.md` — "The run loop" item 1, "After landing", and "PR-Gate".

**Issue:** The stencil is the orch's only procedure after this lands, so every orch follows it literally.
It says to create the task worktree with `lyx fabric add <slug>` before `lyx batten run`, but the batten recipe's Worktree-Create and Seed-Child rows already do that (`contracts/recipes/batten-recipe.yaml:31-39`).
It says to remove the pair with `lyx fabric remove --remote <slug>` after landing, but batten's Worktree-Teardown row does that (`batten-recipe.yaml:49`).
It says Finalize squash-merges; Finalize pushes to main and closes the PR, which ends CLOSED, not merged.

**Fix:** Describe the loop as batten runs it: `lyx batten run <slug>` from the prime, detached, creates the pair, drives the loom and tears the pair down after landing.
Drop the `fabric add` and `fabric remove` steps and state what Finalize does.
Check the rest of the stencil against `batten-recipe.yaml` and `loom-recipe.yaml` the same way.

### [BLOCKING] New tests break PATTERN-test-economy

**Location:** `internal/loomcli/driverstencil_test.go:19-60`; `TestDefaultSkillLoadTimeout` in `internal/shuttleengine/claudeengine`; `TestRunner_CompactedSince_Delegates` (`internal/shuttleengine/session_test.go:120`) and `TestRunner_LoadSkillAndSkillUnknown_DelegateToCapability`.

**Issue:** The PR adds 74 test functions and removes 21.
`driverstencil_test.go` has five tests (ParkMarker, FilesNothing, FrictionFailed, ParentNotice, NoLoomRow), each pinning one phrase of the same stencil: many items of one kind, one test each.
`TestDefaultSkillLoadTimeout` asserts a getter returns a constant: no behavior.
The two delegation tests assert calls into lyx's own code, which `scribe:testing` and PATTERN-test-economy rule out.

**Fix:** Fold the driver-stencil phrase checks into one table test, or into rows of `contracts/stencils/claims_test.go` where the other stencil wording claims live.
Delete the constant-getter test and the delegation tests; cover the behavior they stand for through the public surface that uses it, where no existing test already does.
Go through the other new tests in this PR with the same rule and fold or delete any of the same kind.
