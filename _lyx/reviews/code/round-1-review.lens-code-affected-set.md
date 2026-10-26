```yaml
lens: code-affected-set
outcome: reported
```

Task: list what the diff's changes force outside the lines it touches (lens code-affected-set).

**Per change, the set found**

- `masterAwaitedShellPrefixes` gains `fixPlanCommand`. Covered: `runlevel_test.go` asserts the exact list, and the Master stencil and the websterengine docs are updated.
- A new `State.PlanFixers` field and `PlanFixerState`. Covered: `reclaimEntryTimeStrands`, `RemoveRecoveryStrands` and `reset.go` all handle it (tests in `runlevel_test.go` and `stopper_test.go`). `recordedSessions` leaves it out on purpose (D7).
- A new verb `fix-plan`. `cli.go` registers it, `doc.go` and the overview are updated, and the standalone reed boot (`c.reedUp`) is generic, so it applies. Not covered: the comments at `wiring.go:344` and `batchpause.go:1` (see finding 4). `webster-spec.md` has no verb or conflict-handling text to update, so nothing is missing there.
- A new stencil `webster-template-plan-fix`. Registered in `stencils.go`, and `render.go` renders it. Not covered: the stencil-level tables and render tests (findings 1–3).
- The Master stencil's plan-conflict rung. The awaited-verb mentions are updated; the `fix-verify` mentions at lines 193–216 stay correct as they are.
- `ReadConflictRecord` is added beside `readFindingLines`. Covered: it uses the same `- ` prefix as the writer and `ClaimConflictRoute`.

**Findings**

1. MEDIUM, consistency: `contracts/stencils/stencils.go:536-559`, `roleOpeningStencils`. The map has no entry for the new `plan-fixer` role, such as `"plan-fixer": {"webster-template-plan-fix"}`, although every other spawned role, including `verify-fixer`, has one. The comment above the map says `parentdirective_test.go` enforces the parent and edit directive markers over it. `PATTERN-parent-directive` says the rule is "Enforced by a test over the stencils". With the entry missing, that test never checks the new stencil, so dropping a directive marker from it would pass silently.
2. LOW, consistency: `contracts/stencils/markers_test.go:49-50`, `TestStencils_TopLevelMarkers`. The table lists the verify-fix stencils and the other webster stencils but not `webster-template-plan-fix`. Its required markers (`record_path`, `reply_path`, `worktree_root`, `plan_dir`, `specs_dir`, `edit_directive`) are not pinned there.
3. LOW, consistency: `internal/websterengine/template_test.go:950-970`. The friction-directive on/off cases cover `fork`, `recovery`, `master`, `verify-fix` and `verify-fix-cold`, but not a plan-fix case, although `RenderPlanFixPrompt` has the same `friction.Directive` / `WarnIfMarkerAbsent` path. The two `planfixer_test.go` prompt checks do not cover the friction on/off behaviour.
4. NIT, consistency: stale verb lists. `internal/webstercli/wiring.go:344` says "the verbs that spawn an agent themselves, run, recover-batch and fix-verify", and `internal/webstercli/batchpause.go:1` says "begin-batch, recover-batch and fix-verify". `fix-plan` also spawns an agent and also calls `refuseCovered` (`fixplan.go:87`), and neither comment names it.

Tool calls (3 of 6):
1. `grep -rln "fix-verify"` over the repo, plus a grep in specs, stencils and plugins, to find every place that lists the sibling verb. The manifest held none of these places.
2. `grep` in `batchpause.go`, `wiring.go`, `template_test.go` and `render.go`, plus `sed` on `roleOpeningStencils` and `grep` in `markers_test.go` and `webster-spec.md`. This showed the role map, the marker table and the spec contents, which the manifest lacked.
3. `sed` on `wiring.go` and `template_test.go`, plus `grep` of `refuseCovered` callers, to read the comment, the friction-test table and the pause refusal. The manifest lacked these.
