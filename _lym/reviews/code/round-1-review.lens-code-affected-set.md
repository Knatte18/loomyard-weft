```yaml
lens: code-affected-set
outcome: reported
```

Task: review the diff through the code-affected-set lens. I looked for everything outside the changed lines that reads, asserts or describes what the diff changes.

**Per change, the set found and whether it is covered:**

1. **Refusal text of `sandboxedRoleRefusal`**, now "... outside a hub under its worktree's .lym/harden-sandbox; ..."
   - The only caller is `internal/fabriccli/weft_verbs.go:65`, which passes the text through unchanged. Covered.
   - Tests assert it through `refusalMarker` "from the task session" (`roleguard_test.go:56`). The new text still contains that marker. Covered.
   - `docs/research/single-repo/sandbox-1-setup-2026-11-21/report.md:59` quotes the old text. That is a dated research record, so its staleness is no finding.
   - `internal/configcli/hubwide.go:53` has its own, unrelated refusal. Not affected.
2. **Guard behaviour: any verb passes in the role's own sandbox root.**
   - `internal/fabriccli/roleguard.go` file header and the `sandboxedRoleRefusal` doc comment are updated. Covered.
   - `internal/fabriccli/doc.go`'s new "Sandboxed-role guard" section is added. Covered.
   - **The `internal/fabriccli/fabric.go:6-7` file header is not updated.** It still says the guard refuses every mutating verb from a task session's strand name. Finding F1.
   - **The `doc.go` "# Merge" section (around line 123) is not updated.** It still says "The sandboxed-role guard refuses these verbs from a task session" with only the merge-task exception, but those verbs now pass in the role's own sandbox hub. The new section partly offsets this. Finding F2.
   - `contracts/stencils/webster/webster-body-implementer.md:98-104,146` and `contracts/stencils/weaver/weaver-template-batch.md:97` remain true. An implementer's or weaver's worktree holds no sandbox root, and their "never run a mutating `lym fabric` verb" lines are policy the diff does not change. Covered.
   - `contracts/stencils/harden/harden-rubric-review.md:47` gains a line that agrees with the code. Line 83 ("make none outside the sandbox root") is consistent with it. Covered.
   - `harden-template-seed.md` says nothing about fabric refusals. Not affected.
   - `internal/gatecli/run.go:875-877` strips the strand name for tests and is not affected.
3. **`cloneDestination` extraction.**
   - Callers are `runCloneWithReset` (`clone.go:222`) and `verbTargetDir` (`roleguard.go:36`). The old inline comment's content moved into the new function's doc comment. `fabric.go:104`'s `--into` flag help still matches. Covered.
4. **`sandboxRootRel`.**
   - It is joined from `lymdirs.DotLymDirName` and `lymdirs.HardenSandboxDirName`, the same constants `hardenshed.SandboxRoot` (`internal/hardenshed/sandbox.go:24-25`) and `pairteardown` (`internal/pairteardown/teardown.go:509`) join. No literal repeats it.
   - PATTERN-lym-files (only `lymdirs` names `.lym`) and PATTERN-code-conventions (named constants) are kept. Covered.
5. **Repository rules.**
   - PATTERN-cwd-resolution: the cwd comes through `lymcwd.CwdFrom` and the worktree root through `lymcwd.ResolveWorktree`, with no raw `os.Getwd` or git call. Kept.
   - PATTERN-refusal-way-forward binds only webster, shed and loom, not `fabriccli`. The refusal still names a way forward.
   - PATTERN-test-isolation: `fakeWorktree` is used by one package only, so a test kit is not required.
   - No PATTERN changes are declared, so nothing has to land.
6. **Commit messages.**
   - `e32b6939a` carries `Root-Cause:`, which names the defect the diff removes: the guard judged the strand name only, never the directory. It carries no `Pattern-Departure:` or `Loosened-Check:` trailers, and none are needed. `b6b1bf346` only rewraps comments. Covered.

**Findings:**

- **F1:** LOW, consistency, `internal/fabriccli/fabric.go:7`. The file header still says "The guard refuses every mutating verb from a task session's strand name before the verb runs". The guard now passes any verb whose directory lies under the role's own worktree sandbox root, and it reads paths as well as the strand name. Evidence: `internal/fabriccli/fabric.go:6-7` against `internal/fabriccli/roleguard.go:130-132`.
- **F2:** LOW, consistency, `internal/fabriccli/doc.go` "# Merge" section, the line "The sandboxed-role guard refuses these verbs from a task session." It names the merge-task writer as the only exception, but the diff also lets every merge verb through in a hub under the role's own sandbox root. Evidence: the `doc.go` lines just above the new "# Sandboxed-role guard" section (diff piece 002, context lines 53-56) against `roleguard.go:130-132`.

**Tool calls:**
1. Grepped the repository for the refusal text, `refusalMarker`, `sandboxedRoleRefusal` and guard wording, to find readers of the changed message and the docs that describe the guard; the manifest held only the `fabriccli` files.
2. Read the guard mentions in the webster and weaver stencils, the `doc.go` tail, the `fabric.go` header and the harden stencils, to check whether each statement still holds; these sit outside the manifest.
3. Read `fabric.go` lines 1-12 and the stencil lines about the fabric ban, to confirm the stale header and the stencils' wording; the manifest read `fabric.go` from line 30 on.
4. Searched `internal/testkit` for an existing fake-worktree kit and read PATTERN-test-isolation's kit rule, to check the new test helper against the repository's test-support rule; neither was in context.
