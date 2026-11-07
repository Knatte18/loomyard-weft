```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each change the diff makes, I looked for the callers, tests, docs and rules it forces outside its own lines.

**Set per change**

- **`newAnchorHub` is now a copy of `anchorTemplate`.**
  - Callers are 13 sites in `anchorchange_integration_test.go` plus `newIntegrateHub` (line 828), and all are covered.
  - Every `SetAnchorFailHookForTest`, `killAt` and `SetWeftHeadSHAForTest` call comes after its copy (lines 496–501, 529–534, 565–569, 590–593, 632–636, 657–661, 713–717, 745–748).
  - The site at line 789 still calls `AddPair` for `mover` in the test, as D7 requires.
  - Covered.
- **`newOriginHub` is now a copy of `originTemplate` via `originHubOf`.**
  - Callers inside the diff are covered.
  - Outside the diff: the call-hook subtests in `originreconcile`, `originreconcileended` and `originreconcilemoves` install their hooks after the copy, which is fine.
  - `worktreeseam_integration_test.go` (lines 172, 380, 400, 465) is not covered. See finding 1.
- **`newOriginTaskHub`, `originTaskTemplate` and `buildOriginTask`.** All "task" sites in the diff are converted.
  - The other slug sets (`first`/`second`, `stopped`/`active`/`unrecorded`, `mine`, `plain`, `ghost`) stay in the tests.
  - The build functions touch no `os.Setenv`, `os.MkdirTemp`, global git config or seam setter.
  - Covered.
- **`newMovesPair` is now a copy of `movesTemplate`.**
  - Callers outside the diff: `boltreplayers` line 103, `droplocal` lines 32, 75 and 140, and `worktreeseam` line 491.
  - At `boltreplayers` line 103 the copy comes before `WithoutBoltReplayersForTest`, which is fine.
  - The `droplocal` tests are parallel and run after the serial tests, and the build reaches no replay.
  - `worktreeseam` line 491 is not covered. See finding 1.
- **`endedHub` and `endedTemplate`.**
  - `originBoard` runs on the copy.
  - The `endedHub(t)` callers in `originreconcilemoves` (line 643), `originreconcileended` and the Judge file are all converted.
  - No caller used the old branch return value.
  - Covered.
- **`testing.TB` signature widening** on `originPair`, `writeStatus`, `publishBoard`, `publishBranch` and `boardFileOf`.
  - Other callers pass `*testing.T`, so none breaks.
  - Covered.
- **Rule documents.** `pattern/PATTERN-test-isolation.md` ("Hub fixtures", lines 195–205) already describes registered templates and the seam clause, so no pattern edit is owed by this diff.
  - There is no `Pattern-Departure:` trailer, and none is needed.
  - The range's PATTERN diff is empty, and D7 declares no new entry.

**Findings**

1. **MEDIUM, design.** `TestWorktreeSeam_Events` takes copies after it installs its process-global seam.
   - The seam is installed at `internal/fabricengine/worktreeseam_integration_test.go:185` (`t.Cleanup(fabricengine.InstallWorktreeSeam(seam))`, at the test's top level, so it stays installed for later subtests).
   - The subtests at lines 380, 400, 465 and 491 then call `newOriginHub` or `newMovesPair`.
   - The diff turned both into `hubforge.CopyRegistered` calls, and the first copy of a template runs its build under whatever seam is installed.
   - `movesTemplate`'s build calls `AddPair`, so `TestWorktreeSeam_Events` can run `movesTemplate`'s build under the installed worktree seam. That happens when no earlier test in the binary has built it, for example under `-run TestWorktreeSeam`.
   - This breaks the acceptance criterion "each test that installs a process-global seam takes its copies before installing it", and `PATTERN-test-isolation.md:204`.
   - The summary's claim that the hooks "still follow the copy" is true for the `originreconcile*` hooks and false for this seam.
   - Fix: take every copy of the test before line 185, or build the subtests' hubs ahead of the install.
   - Evidence: `worktreeseam_integration_test.go:166-185,380,400,465,491`; `internal/hubforge/template.go:155-176`.

**Tool calls**
1. A grep of `worktreeseam_integration_test.go` for seam installs, copies and cleanups, and of the other packages for fixture callers, because the manifest did not cover that file's seam install.
2. A grep for template keys, the template var names and the PATTERN text outside the changed files, to look for docs or fixtures that reference them.
