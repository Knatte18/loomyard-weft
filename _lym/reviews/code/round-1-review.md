---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/fabricengine/worktreeseam_integration_test.go:185
    summary: TestWorktreeSeam_Events installs the process-global worktree seam at its top level, then takes copies of registered templates through newOriginHub and newMovesPair, so a template's first build can run under that seam
    fix: In TestWorktreeSeam_Events, take every hub copy the later subtests use before line 185 (t.Cleanup(fabricengine.InstallWorktreeSeam(seam))). Those are the copies behind newOriginHub at lines 380, 400 and 465 and newMovesPair at line 491. Build them before the install and let the subtests read them. The first copy of originTemplate at line 172 already runs before the install and stays. Keep boardkit.InstallReplay at line 490 or move it to the top of the test.
---

### [BLOCKING] A test installs a process-global seam before it takes its template copies

**Location:** internal/fabricengine/worktreeseam_integration_test.go:185 (install), :380, :400, :465, :491 (copies after the install)

**Issue:** `TestWorktreeSeam_Events` installs `fabricengine.InstallWorktreeSeam(seam)` at line 185, at the test's top level, so the seam stays installed for every later subtest.
The subtests at lines 380, 400 and 465 call `newOriginHub`, and the one at line 491 calls `newMovesPair`.
This diff turned both helpers into `hubforge.CopyRegistered` calls, and the first copy of a registered template runs its build (`internal/hubforge/template.go:155-176`) under whatever seam is installed at that moment.
`movesTemplate`'s build calls `AddPair`, which fires the worktree seam's `Made` event.
When no earlier test in the binary has built `movesTemplate`, for example under a narrowed `-run TestWorktreeSeam_Events`, the template's build runs under the fake seam.
The same holds for `originTemplate` when a `-run` filter skips the subtest at line 172, which copies it before the install.
This violates the decision record's acceptance criterion "each test that installs a process-global seam takes its copies before installing it" and the clause at `pattern/PATTERN-test-isolation.md:204`.
The decision record's own constraint on `PATTERN-test-isolation` makes this a listed-constraint violation, so it is BLOCKING however small its effect on a full-package run.
`_lym/landing/summary.md` says the hooks "still follow the copy"; that holds for the `originreconcile*` hooks and not for this seam.
The file is outside the diff's lines, but the diff's change to the two helpers is what makes it a violation.

## Rejected

- Seam and hook ordering in `anchorchange_integration_test.go` (`killAt`, `SetAnchorFailHookForTest`, `SetWeftHeadSHAForTest`): every install comes after its copy, as the code-correctness and code-tests lenses read.
- Seam and hook ordering in `originreconcile_integration_test.go`, `originreconcileended_integration_test.go` and `originreconcilemoves_integration_test.go` (`SetOriginCallHookForTest`, `SetOriginVerdictHookForTest`): every install comes after its copy.
- `boltreplayers_integration_test.go:103`: `newMovesPair` runs before `WithoutBoltReplayersForTest`, so no build runs under an emptied Bolt replayer registry.
- Build functions calling `os.Setenv`, `os.MkdirTemp`, `git config --global`, a seam setter or a once-only registration, or keeping the TB past the build: none does, as read in every `Build` and its helpers.
- Missing coverage: the diff adds no test and changes no assertion, and every changed start state is pinned by its consuming tests, per the code-tests lens.
- Comment wording that names other template symbols, such as `originTemplate's hub` in the comment on `endedTemplate`: the rubric defers comment line breaks to the comment lint, and the surrounding test code names sibling symbols the same way, so no repository convention is broken.
