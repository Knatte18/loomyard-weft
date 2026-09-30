# Card 15 — run-exit-dispositions

**Edit:**
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`

**Uses:**
- `internal/websterengine/audit.go`
- `internal/websterengine/auditledger.go`
- `internal/websterengine/summary.go`

**Intent:**
Apply D8 at run exit: the whole-session cross-check passes every finding a record-batch already dispositioned, warns on a policy finding nobody dispositioned, and demotes `done` to `stuck` on an undispositioned correctness finding, naming each suspect path.

`runExitAuditCrossCheck` returns `(warnings []string, stuckReason string, err error)`:

- The nil-`ForkAudit` refusal and the audited-count check are unchanged and stay errors.
- Findings are built with `CheckParent` (identity from `result.SessionID`) and `CheckFork` over every fork, then every dispositioned identity is dropped.
- Policy findings are recorded as run-level warnings (`recordRunWarning`) against a state reloaded under `AcquireStateMutation` and saved before release; their texts are returned as `warnings`.
- Correctness findings produce a stuck reason: `run-exit audit found N correctness finding(s): <details>; suspect paths: <paths>; way forward: revert or re-derive the named paths on the warp with git, then re-step the Webster row (lyx webster run)`.
  They are not dispositioned: the next run's fresh Master session carries none of them.
- A `ClassifyViolation` probe error is returned as an error.

`mapMasterDone` threads the warnings onto `RunResult.Warnings` and demotes a `done` outcome to `outcomeStuck` with that reason, the same way the integration stage demotes a regression.
After the integration stage, `Run` reloads state and, when `RecordedAuditWarnings` is non-empty and summary.md exists, calls `AppendAuditWarnings` with them; a missing summary on a non-done outcome skips the section.
Update `Run`'s and `runExitAuditCrossCheck`'s doc comments.

Tests (integration) in `runlevel_test.go`:

- The named-spawn scenario end to end: begin and record batches 2 and 3 with the finding present from batch 2 on, then a `done` Master whose whole-session audit carries it → `Run` returns `done`, no error, and no second warning for the finding.
- A `fabric-reference` in the integration fork's transcript on a `done` run → outcome `done`, one run-level entry in state.json's `auditWarnings`, the line on `RunResult.Warnings` (what `lyx webster run` prints), and a `## Audit warnings` section in summary.md.
- A Master `parent-write` into a tracked file after the last batch → outcome `stuck`, the stuck reason names the path and `git`; the test then restores the file with `git checkout`, runs `Run` again with a clean audit, and gets `done`.
- Rewrite `TestRun_DoneWithParentWriteViolationIsHardError` to the stuck demotion.

**ImpactSummary:** Run's exit check now warns or demotes instead of erroring on audit findings; Webster's shed adapter already maps both outcomes and needs no change.
