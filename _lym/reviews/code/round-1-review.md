---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabriccli/clone.go:131
    summary: The reorder does not change what the render reads, so the diff does not show that a clone now leaves the prime rendered
    fix: Settle the cause the Goal calls unconfirmed, then make the change that follows from it. The render reads only the heddle store in the Bolt checkout and the shipped layer, never the per-worktree module configs, so firing after the module-config commit leaves its inputs as they were. Add a tier-1 subtest in internal/fabricengine/worktreeseam_integration_test.go that installs the real heddleengine.Seam, clones with fabriccli.CloneAndWire and asserts that the prime's anchor holds the rendered heddle files. Move the event again only if that test fails first, for example after the hub walk that seeds the board defaults. Correct the comment at clone.go:131-134 and the sequence claim in fabricengine/doc.go to say what the test proves.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabriccli/clone.go:144
    summary: A hub walk failure after a failed render drops the seam warning, so it reaches only the log
    fix: In CloneAndWire, when hubreconcile.Ensure returns an error and res.SeamWarning is non-empty, return an error that carries the seam warning text as well, so the errWithRecord envelope prints it. Keep the clone intact as now. Correct the comment at clone.go:133, which says a walk failure still leaves the prime rendered, to say that holds only when the render succeeded.
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/heddleengine/seam.go:56
    summary: Render warnings that leave the prime unrendered are logged only, so a clone that renders nothing prints an empty seam_warning
    fix: In Seam.render, return an error naming the Report.Warnings when a made event's render wrote none of its targets because of them (a blocked temp dir, a target outside the exclude list, an unmarked existing file), so fabric's seamWarning carries them with the render verb. Leave the TookIn path's behaviour as it is unless the same change covers it. Add a row to heddleengine's seam test for a Made event whose render only warns, asserting the returned error names the warning.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/worktreeseam_integration_test.go:170
    summary: No test pins the made event's position in the clone sequence, after the module-config commit and before the hub walk
    fix: In the clone made-event subtest, have the fake seam record whether the module-config commit is in the weft history and whether the hub walk has run at the moment Made fires, and assert both. Add a row for a walk failure, forced through the hubreconcile test hook or a refusing board remote, that asserts the made event was heard before the clone failed. If F1 moves the event, assert the new position instead.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabriccli/clone.go:253
    summary: The seam_warning field of the lym fabric clone envelope has no test, and neither does the empty value on a healthy clone
    fix: Add a case that runs the clone verb through runCloneWithReset, in the existing fabriccli clone envelope test if one asserts fields, otherwise beside the pull envelope check in worktreeseam_integration_test.go. With a failing seam assert the output carries seam_warning naming lym heddle render. With a healthy seam assert it carries seam_warning as an empty string. Fold the healthy-clone assertion into this case, not into a second test.
---

### [MEDIUM] The reorder is not shown to change what the render reads

**Location:** internal/fabriccli/clone.go:131
**Issue:** The Goal wants a clone to leave the prime rendered and names two suspects, one of them the ordering against the module-config commit. `Renderer.Render` reads `r.read(req.BoltDir)`, the heddle store and shipped layer, and the per-worktree module configs are not among its inputs. The diff moves the call on the strength of the unconfirmed suspect and adds no assertion that the prime ends up rendered. The success path stays untested, and the only observable change is the new `seam_warning` output.

### [MEDIUM] A walk failure after a failed render drops the warning

**Location:** internal/fabriccli/clone.go:144
**Issue:** The event now fires before `hubreconcile.Ensure`. When the walk then fails, `CloneAndWire` returns `CloneResult{}`, so `SeamWarning` is lost, and the failing path prints through `errWithRecord`, which has no `seam_warning` field. A failed render on that path is again only in the log, which the Goal says must not happen. The comment at line 133 overstates the guarantee.

### [MEDIUM] Render warnings leave the prime unrendered without reaching the operator

**Location:** internal/heddleengine/seam.go:56
**Issue:** `Seam.render` logs `Report.Warnings` and returns only the render's error. A blocked temp dir, a file target missing from the exclude list or an existing file heddle did not write all leave the prime unrendered with no error, so the clone prints an empty `seam_warning`. This is the Goal's symptom through the other door: the clone succeeds, the prime has no files, and the operator is told nothing.

### [MEDIUM] The made event's position in the clone sequence is unpinned

**Location:** internal/fabricengine/worktreeseam_integration_test.go:170
**Issue:** Commit b536821a removed the module-config assertion, and the remaining clone assertions check root, Bolt checkout, exclude lines and paths only. Moving the call back above the module-config commit, or after the walk, would pass every test, so the ordering this change exists to make has no coverage.

### [MEDIUM] The clone envelope's seam_warning field is untested

**Location:** internal/fabriccli/clone.go:253
**Issue:** The new subtest drives `CloneAndWire` and so bypasses `runCloneWithReset`. Dropping `"seam_warning": res.SeamWarning` from the envelope map, or leaving the field stuck non-empty on a healthy clone, passes every test. The envelope field is how the Goal's printed warning reaches the operator.

## Rejected

- Lens code-tests, LOW finding that a healthy clone's empty `SeamWarning` is unasserted: the same case as F5's envelope assertion, folded into F5 rather than kept as a second finding.
- Lens code-correctness, note that errors before the new call site leave the prime unrendered: the clone fails on those paths and its operator completes the wiring with `lym fabric reconcile`, so the prime is not a successful clone's output; the old call site had the same property for every earlier failure.
- Lens code-affected-set raised nothing; its checks of the `RegisterTeardown` rename, the docs, the PATTERN entries and the commit messages found no defect, and I read the same hunks and agree.
