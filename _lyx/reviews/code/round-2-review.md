---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/webstercli/restoreplan.go:19
    summary: restore-plan keeps the operator audience although the plan fixer's prompt now tells a spawned role to run it
    fix: In internal/webstercli/restoreplan.go change the Annotations value of restorePlanCmd from clihelp.AudienceOperator to clihelp.AudienceRole, as rebaseline.go already does for the sibling verb the same prompt names, then check that no test or generated command index pins restore-plan under the operator audience.
---

Both focus questions settle as not a defect.

- `internal/websterengine/runwrites_test.go` seeds `PlanFixers: []*PlanFixerState{{SessionID: "p1"}}` beside distinct verify-fixer and recovery sessions.
  It asserts the exact audited list `s1, s2, r1, r2, v1` and the recoveries and forks derived from it, so `p1` appearing in either `recordedSessions` or `loadRunWrites` would fail it.
- `contracts/stencils/stencils.go` maps `"plan-fixer"` to `webster-template-plan-fix` in `roleOpeningStencils`, which `parentdirective_test.go` walks for both the parent and edit markers.
  `markers_test.go` also pins the stencil's top-level markers.

The rest of the diff reads as the amended entry asks.
The bounds (same findings set, two per batch), the attach on a non-terminal last record, the HEAD and new-dirty-path overrule, the reclaim on run entry and the stop on reset all match D6 and D7 and are covered by `planfixer_test.go`, `runlevel_test.go` and `stopper_test.go`.

### [LOW] restore-plan stays operator-audience while a role prompt names it

**Location:** internal/webstercli/restoreplan.go:19
**Issue:** `AudienceRole` is documented as "a command a spawned role agent runs, named by that role's prompt", and the command index lists a command only under its own audience.
`contracts/stencils/webster/webster-template-plan-fix.md` tells the plan fixer, a spawned role, to run `lyx webster restore-plan` twice, but `restorePlanCmd` is annotated `AudienceOperator`, so the role index omits it.
`rebaseline`, the other verb that prompt names, is already `AudienceRole`.
