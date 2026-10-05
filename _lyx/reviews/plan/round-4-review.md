---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/17-orch-role-run-loop.md (run-loop bullet)
    summary: The run-loop bullet keeps `lyx batten pause` and `lyx batten goto`, which leave the batten run paused, and states no resume step or claim for them
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/17-orch-role-run-loop.md (Uses, After landing bullet)
    summary: After landing names the friction notes directory, but Uses holds no entry for its accessor `internal/loomengine#LoomFrictionDir`, while the drive-reports directory is grounded by `internal/shedrun#DriveReportsDir`
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/20-drop-getter-delegation-tests.md (TestRunner_ContextTokens_Delegates bullet)
    summary: The reason for keeping the `ContextTokens` delegation test applies word for word to the deleted `CompactedSince` test, so it does not separate the two
---

### [MEDIUM] Batten pause and goto name no resume step

**Location:** `_lyx/plan/17-orch-role-run-loop.md`, the "The run loop" bullet.
**Issue:** The card rewrites the run loop and keeps step 3 of the current stencil, which has the orch hold a run with `lyx batten pause <slug>` and send it to a producer with `lyx batten goto <slug> --to <producer>`.
`internal/battencli/cli.go` describes `goto` as moving a halted lifecycle "onto a named row, paused", and `run` as "start or resume".
The same card applies the paused-needs-resume check to `lyx loom goto` and `lyx shed goto`, but not to these two verbs.
After a batten `pause` or `goto` the stencil gives the orch no way to continue the run, and the `orch-template-role.md` row has no must claim for it.
**Fix:** In the run-loop bullet, state that `lyx batten pause` and `lyx batten goto` leave the batten run paused and that the orch resumes it with `lyx batten run <slug>`.
Add a must claim for that resume to the `orch-template-role.md` row.

### [LOW] Friction notes directory has no Uses entry

**Location:** `_lyx/plan/17-orch-role-run-loop.md`, `Uses` and the After landing bullet.
**Issue:** The bullet says "the friction notes directory and the run's `drive-reports` directory ... both under the prime's durable lyx directory".
Only the second is grounded in `Uses` (`internal/shedrun#DriveReportsDir`).
The first is `internal/loomengine#LoomFrictionDir`, which `LoomFrictionDir`'s own comment places under the durable tree, so the claim matches `PATTERN-durable-vs-ephemeral-state` but its source is not cited.
**Fix:** Add `internal/loomengine#LoomFrictionDir` to `Uses`, and cite it beside `DriveReportsDir` in the bullet.

### [LOW] The reason for keeping the ContextTokens test does not separate it from CompactedSince

**Location:** `_lyx/plan/20-drop-getter-delegation-tests.md`.
**Issue:** The card keeps `TestRunner_ContextTokens_Delegates` because "the engine's own `ContextTokens` tests call the engine directly, so it is the only test of the Runner returning the engine's reading".
That is true: no other test in `internal/shuttleengine/session_test.go` calls `Runner.ContextTokens` on a success path.
The same holds for the deleted `CompactedSince` test, whose engine tests also call the engine directly, and both Runner methods have the same three-line shape (`toldErr`, `sessionCycler`, delegate).
The success path through `sessionCycler` is already covered by the `SessionIdle`, `ClearSession` and `CompactSession` tests.
The card's own rule is "a pass-through keeps no test of its own", so keeping one of two identical pass-throughs "by choice" is inconsistent.
**Fix:** Delete `TestRunner_ContextTokens_Delegates` as well, state in the card that no pass-through keeps a test, and remove `cyclerEngine`'s `tokens` and `known` state with it, leaving `ContextTokens` as a zero answer.
Update the `ImpactSummary` to match.
