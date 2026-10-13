---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-loom-fmt-verb.md:33
    summary: planGoFormat's unreadable-file refusal has no named input and no TestFmtCmd row that reaches it
    fix: In card 6's Tests list under TestFmtCmd's rows, add a row naming a reachable unreadable input beside a misformatted file in the same call, for example a directory named dir.go inside the worktree, which passes EvalSymlinks, the inside-root check and the .go suffix check and then fails os.ReadFile on every OS. Assert that the call refuses whole, that refused names dir.go with its read error as the reason, and that the misformatted file is left unwritten. In the planGoFormat paragraph, also state that only a not-exist failure of filepath.EvalSymlinks is refused as no such file, and that any other EvalSymlinks failure is refused with its own error as the reason.
---

### [MEDIUM] The unreadable-file refusal is unreached by any test row

**Location:** `_lyx/plan/06-loom-fmt-verb.md:33`

**Issue:** Card 6 lists six reasons that `planGoFormat` returns a `refused` entry for: a path that does not exist, a path outside the worktree, a symlink resolving outside it, a non-`.go` path, an unreadable file and one that does not parse.
`TestFmtCmd`'s rows reach five of them: nonexistent, outside, symlink, non-`.go` and unparsable.
None names an input that reaches the unreadable-file branch, which is the `os.ReadFile` failure after a path has passed the `EvalSymlinks`, inside-root and `.go` checks.
That branch is reachable, for example by a directory named `x.go` or a mode-0 file when the test is not running as root, so this is an untested refusal branch, which the rubric's reachability rule names as a finding.
The card also derives the `no such file` reason "from the `filepath.EvalSymlinks` failure" without saying what any other `EvalSymlinks` failure, such as a permission error on a parent directory, is refused as.
As written, a fixer could label every such failure `no such file`, which would mislead the agent the envelope answers.
Round 1's F2 named both the nonexistent path and the unreadable file as unreached.
Its fixer report fixed only the nonexistent half, adding that row, and left the unreadable branch without a row, so this is the remainder of that finding rather than a new one.

## Focus directives settled

- Card 6, pre-run path: not a defect.
  `resolvePersistentPreRun` (`internal/loomcli/cli.go`) arms every loom verb except `loom`, `checklist` and the review groups, and `resolveRunID` (`internal/loomcli/arm.go`) takes `args[0]` as the run-id unless the verb is `reject`, `insert-card` or `remove-card`.
  `resolveRecipe` then calls `shedrun.ReadSeed`, whose `ValidateRunID` refuses a separator.
  Card 6 adds `fmt` to that exclusion list, so `c.runID` stays `shedrun.SelfRunID` and a path argument such as `internal/x/y.go` reaches `RunE`.
  `TestResolveRunID`'s new row passes `internal/x/y.go`, a path with a separator, and pins `wantRunID` to self.
  That is the same depth at which the existing `reject`, `insert-card` and `remove-card` rows pin their exclusion.
  The rest of `armAt` on the lightweight path is shared with `lint-comments`, which `lintcomments_integration_test.go` drives end to end.
  `c.cwd` is set by `wireLightweight` to `location.AnchorPath()`, which `lyxcwd.Resolve` proves equal to the invoking cwd, so resolving relative paths against `c.cwd` matches the card's claim.
- Card 6, refusal reachability: the nonexistent path is reached by its own row, and the reason is ordered correctly before the inside-root check.
  The non-`.go`, outside, symlink-outside and unparsable reasons each have rows, but the unreadable file does not; see F1.
- Card 7, leading recorder: pinned.
  The card adds `{"removeWorktreeLeftover", "func removeWorktreeLeftover(rec *Mutations, "}` to `destructiveGuardRecordingExecutors`.
  That string matches its planned declaration `func removeWorktreeLeftover(rec *Mutations, container, target string, seam leftoverSeam) error` in `destroy.go`, which is the file `TestMutationRecord_FabricengineProductionSource` reads.
  The per-OS `latewriter_*.go` files only call `errors.Is` and carry none of `TestNoDestructiveBypass_FabricengineProductionSource`'s banned tokens.
  The removal itself stays in `destroy.go` through `removeContainedPath`.

## Rejected

- Card 6's `os.WriteFile` in `internal/loomcli`: `TestNoUncontainedWrite_FabricengineProductionSource` scans `internal/fabricengine` only, so the new verb trips no write scan.
- Card 7's `TestRemove_RefusesForeignWorktreeWithoutDeletingIt` seam: `hubforge.Hub.Topology` is a `*fabricengine.Topology` built per hub by `fabricengine.NewTopology`, so setting a seam on it per instance holds under `t.Parallel`.
- Card 7's Linux failure mechanism: git's worktree removal deletes the registration even after the work-tree delete fails, which matches D7's premise, so a read-only subdirectory holding a committed file leaves the target unregistered, clean and on disk, as the rows require.
