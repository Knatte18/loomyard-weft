---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: PATTERN.md:7
    summary: >-
      Six entries other than PATTERN-sparse-hub-base carry shortened clauses against the merge base, an undeclared change to PATTERN-cwd-resolution, PATTERN-config-layers, PATTERN-review-round, PATTERN-friction-capture, PATTERN-testkit and PATTERN-hubforge-fixtures
    fix: >-
      In PATTERN.md restore the six entry lines byte for byte to their merge-base text (`git show 90f613277e539c98eb07698ed58b50f705e99a77:PATTERN.md`), so only PATTERN-sparse-hub-base differs from the merge base; the clauses to restore are cwd-resolution "; nothing else calls `os.Getwd` or `git rev-parse`", config-layers ", machine layer included", review-round ", end on a verdict", friction-capture "reflected at the run's end, archived once covered", testkit ", never duplicated", hubforge-fixtures " or wrapped". Do not cut any other entry to make room; if `lyx loom lint-pattern` then reports growth past the headroom line, report that in the fixer report as a dispute for the operator rather than trimming an undeclared entry.
  - id: F2
    severity: BLOCKING
    class: design
    location: PATTERN.md:14
    summary: >-
      The PATTERN-sparse-hub-base entry line does not carry the declared text; it drops the per-worktree repo default scope, the seed-only exemption, the never-a-template-fill clause, and the walk-strip and lenses clause
    fix: >-
      In PATTERN.md replace the entry's requirement text with the declared text, keeping the `PATTERN-sparse-hub-base` name, the `—` separator and the `[background](pattern/PATTERN-sparse-hub-base.md)` link - "Writing a hub's base file (a hub-wide module's hub file or a per-worktree repo default, not seed-only): only the units the hub tuned, never a template fill; only `hubreconcile`'s one-time walk strip drops a unit the template holds, bar `lenses`' template-equal entries." Leave pattern/PATTERN-sparse-hub-base.md as it is.
  - id: F3
    severity: BLOCKING
    class: design
    location: internal/loomcli/smoke_resumearming_test.go:30
    summary: >-
      TestLoomResumeReconcilesThenRefusesWithoutLiveDriver fails its TempDir cleanup because the resume it runs now commits the board marker and a detached push outlives the test and races removal of the bare remote
    fix: >-
      In TestLoomResumeReconcilesThenRefusesWithoutLiveDriver replace the `runLoomCLINoFatal(exe, worktree, 60*time.Second, "loom", "resume")` call with `runLoomCLIWithEnvNoFatal(exe, worktree, append([]string{"FABRIC_SKIP_PUSH=1"}, skipBoardPush...), 60*time.Second, "loom", "resume")`, with a comment above it that the reconcile commit fires a detached push that would outlive the test and race TempDir's removal of the bare remote, and that the test asserts the commit and never the push, as `TestLoomResumeReconcilesTheHubConfigBeforeArming` in smoke_prebootstrap_integration_test.go does. Then check the other tmux-tier tests in internal/loomcli that run `loom resume`, `loom start` or `loom step` through `runLoomCLINoFatal` on an unstamped hub (smoke_drivingsurface_test.go, smoke_driverstrand_test.go, smoke_bootstrapwiring_test.go) and give each that runs a walk the same environment.
---

### [BLOCKING] Six undeclared PATTERN entry changes

**Location:** PATTERN.md:7, PATTERN.md:19-22, PATTERN.md:70, PATTERN.md:79, PATTERN.md:117-118
**Issue:** `git diff 90f61327..HEAD -- PATTERN.md` changes `PATTERN-cwd-resolution`, `PATTERN-config-layers`, `PATTERN-review-round`, `PATTERN-friction-capture`, `PATTERN-testkit` and `PATTERN-hubforge-fixtures`, and the declared changes list names none of them.
The last commit (`8b07cacf`, "cut PATTERN.md back under its headroom line") made these cuts to bring the overview back to the merge base's size, which the same branch's first fix (`3ab5c4da`) had already had to undo once.
The round-1 fix restored ten entries and the later trims removed clauses from six of them again.
Most dropped clauses still exist in the background files, but the entry line is the invariant agents read inlined, and a clause that leaves it weakens the rule for them, for example `PATTERN-hubforge-fixtures` no longer forbids wrapping `hubforge` in its own fixture type at the overview.
The focus question is answered no: more than `PATTERN-sparse-hub-base` changed.
The ten entries the focus names are otherwise the same as at the merge base, bar `PATTERN-friction-capture`, `PATTERN-testkit` and `PATTERN-hubforge-fixtures`, which are among the six.

### [BLOCKING] PATTERN-sparse-hub-base does not land its declared text

**Location:** PATTERN.md:14
**Issue:** The declared change adds the entry with a scope, an exemption and an exception clause.
The landed line reads "Writing a hub's base file: only the units the hub tuned.", and the background file carries the rest.
The rubric requires each declared entry to land with the declared text, so this is a missing landing, not a style choice.
The last commit shortened it from a version that still had "never a template fill".

### [BLOCKING] Publish failure: TestLoomResumeReconcilesThenRefusesWithoutLiveDriver

**Location:** internal/loomcli/smoke_resumearming_test.go:30
**Issue:** The log (`.lyx/verify/verify-5.log`, lines 68-72) shows `TempDir RemoveAll cleanup: unlinkat .../weft-bare/objects/tmp_objdir-incoming-...: directory not empty`, which is a git push still writing into the bare remote while the test's cleanup removes it.
The merge-in commit does not cause it: this branch's walk (`internal/hubreconcile/walk.go`) now treats an absent board marker as a stale stamp, writes the marker and commits it, and `reconcileHubWide` spawns the detached board sync whether or not it committed.
`boardengine.SpawnSync` returns early only under `testing.Testing()`, which the real `lyx` binary the test runs is not, so the push runs in the background of a binary the test never waits on.
The sibling test `TestLoomResumeReconcilesTheHubConfigBeforeArming` already avoids this with `FABRIC_SKIP_PUSH=1` and `skipBoardPush`; the failing test runs the same verb without them.

## Focus answers

- PATTERN versus the merge base: no, six entries beyond `PATTERN-sparse-hub-base` changed (F1), and `PATTERN-sparse-hub-base` itself lacks its declared text (F2).
- `CloneAndWire` in `internal/fabriccli/clone.go`: not a defect.
  When `b.Commit` fails after the marker is written, the code removes the marker (`os.Remove(markerPath)`) only when the board was seeded from nothing, which is the only case in which the clone wrote it.
  Coverage exists as a late section of `TestRunCLI_CloneEndToEnd` in `internal/fabriccli/cli_test.go` (the pre-commit hook rejecting the marker, lines 215-243): it forces the commit failure, asserts a non-zero exit, and asserts the marker file is absent.
