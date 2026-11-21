```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I named the test that pins it, or none.

**Behaviours and the tests that pin them**

1. **A process whose cwd sits under the link-resolved root is now reaped** (`internal/hardenshed/reap.go:197-205`).
   - Pinned by `TestReap/EndsAnOrphanWhoseCwdIsUnderTheRootsLinkTarget` (`internal/hardenshed/reap_test.go:115-137`).
   - On the merge base `EndProcess` is never called, so this test fails there.
   - The process's `Cwd` is built with `filepath.EvalSymlinks(root)` (reap_test.go:125-129), which is the same call the code makes. Only the input is built that way: the expected value is the literal `[]int{12}`. Resolving in the test also keeps the input right when the temp dir itself sits under a link, such as `/var` → `/private/var` on macOS. A hand-joined `linkTarget` path would break there, so this is not a tautology finding.
   - End to end it is pinned on Linux by `TestSandboxVerbs`, subtest "clear ends a process naming the root…" (`internal/hardencli/cli_integration_test.go:125-173`, tier 1). The orphan now has `Dir = root` and no root on its command line, and the test asserts the process exits and is reported in `ended`.
2. **The written root still matches** (the `roots` slice starts with `deps.Root`).
   - Pinned by `TestReap/EndsRecordedProcessesThroughEndAndUnrecordedOnesThroughEndProcess` (reap_test.go:82-113, PIDs 11 and 12 under a root with no link). `LeavesTheCallersAncestors` and `MarksWhatAStartReapFindsAsInterrupted` pin it too.
3. **The resolved form is added only when it differs from the written root** (`resolved != deps.Root`).
   - Only the existing tests over a root with no link pin it. The duplicate it prevents would be harmless, because `slices.ContainsFunc` deduplicates the result, so no test needs to pin it.
4. **When `EvalSymlinks` fails, as when the root is missing, only the written root is compared** (reap.go:198-200). The summary claims this behaviour.
   - **No test pins it.** Every `TestReap` row builds an existing root, at reap_test.go:63-64, 117-123, 203 and 213. No row runs with a missing root and a listed process.
5. **A process under the link target but outside the sandbox root is not matched through the resolved form.**
   - **No test pins it.** The new subtest lists only the matching PID 12 (reap_test.go:129). The boundary negative `root + "2"` (reap_test.go:90) is checked against the written root only.
6. **Doc and comment wording** (`doc.go:64-65`, the `rootProcesses` doc comment). This is static prose, so it is exempt.

**Findings**

- **MEDIUM · design · `internal/hardenshed/reap_test.go:115` (`TestReap`)**
  - Summary: no test pins the fallback where `EvalSymlinks` fails and only the written root is compared.
  - Why it matters: a regression such as flipping `err == nil` to `err != nil` would add `""` to `roots`. `namesRoot(process, "")` matches every absolute path, because `strings.Index(text, "")` is 0 and `text[0]` is `/`. A reap over a missing root would then end every unprotected process on the machine, and no tier-0 row would fail.
  - Evidence: `internal/hardenshed/reap.go:197-200`, `reap.go:235-247`, and the rows listed above, none of which has a missing root.
  - Fix: add a row to `TestReap` that runs `Reap` over a root that does not exist. It lists one process under that root and one elsewhere, and asserts that `EndProcess` gets only the first PID.
- **LOW · design · `internal/hardenshed/reap_test.go:129`**
  - Summary: the new link subtest has no negative row, so nothing shows the resolved-root match stops at the root's boundary.
  - Examples of the missing row: a process whose `Cwd` is `resolved + "2"`, or `linkTarget` itself, outside `harden-sandbox`.
  - Evidence: reap_test.go:129 lists only PID 12. The boundary row at reap_test.go:90 runs against a root with no link.
  - Fix: add such a process to the new subtest's `processes` and keep `want` at `[]int{12}`.

**Tool calls**

1. I grepped the subtest names, `processes`/`Cwd` rows and `tiergate` lines in `reap_test.go` and `cli_integration_test.go`, and the base orphan line through `git show 13cebe3d`. My context held only the beginning of `reap_test.go` and the new subtest, not the other rows or whether any of them ran with a missing root.
