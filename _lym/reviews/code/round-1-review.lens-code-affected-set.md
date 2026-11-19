```yaml
lens: code-affected-set
outcome: reported
```

**List: each change and the set of things that read it**

- `lymcwd.OneRepository` and the root-marker read were removed, with `ResolveWithin`/`readRecordedAnchor` back to two-repository rules. Readers: battencli/loomcli seams (removed), the PATTERN-cwd-resolution entry and background (updated to the declared text exactly), and `TestReadRecordedAnchor_MarkerSemantics` (rows removed). Covered.
- The `statuscommit` operation wait and the Stalled/Protected seams were removed. Readers: battencli and loomcli `commitStatusDeps` and their seam docs, the `statuscommit` package doc, the docs/overview.md statuscommit line (restored to "skip while mid-merge"), and the PATTERN-run-fsm entry and background. The background cites `TestNew_OwedPushRetriesBehindTheGate`, which exists at `internal/statuscommit/statuscommit_test.go:257`. Covered.
- `MidMergeRecordsUnsettled` was removed. Readers: loomcli `start.go` (`ownMergeInLeftover`, `refuseOverUnfinishedMerge`), landingshed `finalize.go` and `premerge.go`, and their tests. A grep for "unsettled" finds no fabric or landing leftover. Covered.
- `runfsm.Inputs/Verdict.PushOwedWayForward` was removed. Readers: pairteardown `runread.go` and its test, the `runfsm` act test, `runfsm/doc.go:7-9`, `pairteardown/doc.go:203` (both neutral), and the PATTERN-run-fsm background ("names `lym fabric push` in the owed worktree"). Code, tests and docs are covered. The change description is not: see F1.
- `Fabric.ChangedCodePaths` and the `Deps.ChangedCodePaths` seam were removed. Readers: loomcli `landingdeps.go`, envkit, both seam-allowlist tests, landingshed `deps.go`/`doc.go`/`finalize.go` and `finalize_test.go`. Covered.
- `gitrepo.MergeStartNoFastForward` and `mergeStart` were removed. Readers: `gitrepo/doc.go` and the pinned-method list in `cmd/lym/gitrepoboundary_test.go` (`mergeStart` → `MergeStart`, which matches f06acf39^). Covered.
- The deleted files `recordsduringmerge.go`, `recordspaths.go` and `clone_onerepository.go` had allowlist rows in `cmd/lym/destructiveguard_test.go` and `uncontainedwrite_test.go`. Those rows are removed. Covered.
- The `--single-repo` flag, `BoltBranchName` and the clone help were removed. Readers: fabriccli `fabric.go`, `clone.go`, `doc.go` and `cli_test`, README.md, and docs/getting-started.md. Covered.
- Deferred failed-push mail: `fabricengine/doc.go:1307` and `:1552` still say a deferred push "mails nothing". Consistent.
- PATTERN changes the decision record declares:
  - The `PATTERN-run-fsm` and `PATTERN-cwd-resolution` PATTERN.md lines match the declared text verbatim. The backgrounds lose their one-repository text.
  - The `PATTERN-single-repo-hub` entry and its file are both removed.
  - The edits to fabric-boundary, lym-files and pair-remote drop only one-repository paragraphs.
  - Landed.
- Leftover grep (cited test names, `MergeOperation`, `PushOwedWayForward`, "records unpushed", `` `git push` in the prime ``): no stale reference in code, docs, stencils or pattern/. The one hit, `originreconcile_integration_test.go:398`, is the prime way-forward test, which predates this change.
- Commit messages: each claim in 4dcf7cd7's message ("Kept…", "Reverted widenings…", "ChangedCodePaths… removed", "PATTERN… removed") matches the diff.

**Findings**

- F1. LOW, class consistency, at `_lym/landing/summary.md:28`.
  - The change description says `FailedPushWayForward` "predates f06acf39 and stays with all its callers". The diff removes one of those callers: pairteardown's `inputs.PushOwedWayForward = fabricengine.FailedPushWayForward(owed[0])`.
  - Evidence:
    - Diff 025, `internal/pairteardown/runread.go:239` (removed line).
    - `git grep` shows the call at 7346d6a8 `internal/pairteardown/runread.go:239` and absent at HEAD.
  - Fix: say that pairteardown's call goes with `runfsm`'s `PushOwedWayForward` (D5, `operator_unpushed`'s way forward), and that the other production callers remain.

Tool calls:
1. Grep for stale references to the removed test names, seams and wordings, plus the "Held by" lines in the pattern backgrounds. The manifest's leftover grep did not cover cited test names or way-forward wording.
2. Grep the `runfsm`/`pairteardown` package docs, the fabric doc's deferred-mail lines and the statuscommit docs. These docs describe the changed behaviours, and the diff pieces did not show them.
