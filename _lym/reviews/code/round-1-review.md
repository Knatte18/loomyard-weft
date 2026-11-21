---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/hardenshed/reap_test.go:115
    summary: No TestReap row pins the two other outcomes of the new root-resolution branch in rootProcesses, the written root still matching under a linked root and the written root alone when the root does not resolve.
    fix: "In TestReap in internal/hardenshed/reap_test.go, extend EndsAnOrphanWhoseCwdIsUnderTheRootsLinkTarget with a second process whose Argv names the written root (filepath.Join(root, \"hub-one\")) and whose Cwd lies outside both roots, and set want to both pids. Add one subtest that runs Reap over a root that does not exist (SandboxRoot of a fresh t.TempDir with no .lym created), listing one process whose Cwd is under that written root and one whose Cwd is elsewhere, and assert Reap returns no error and EndProcess gets only the first pid."
    origin: lens:code-tests
---

### [MEDIUM] The written-root and unresolved-root outcomes of root resolution have no test row

**Location:** internal/hardenshed/reap_test.go:115

**Issue:** `rootProcesses` now has three outcomes: the resolved root added beside the written one, the written root kept after resolution, and the written root alone when `filepath.EvalSymlinks` fails (`internal/hardenshed/reap.go:197-200`).
Only the first is pinned, by `EndsAnOrphanWhoseCwdIsUnderTheRootsLinkTarget`.
Every other `TestReap` row builds an existing root with no link, so `resolved == deps.Root` there.
A change that replaced the written root with the resolved one would pass every row, yet production roots always sit under the `.lym` link (`internal/loomcli/hardenwiring.go:184`), so a process naming the root on its command line by the `.lym` path would escape the reap again.
The unresolved-root fallback, which `_lym/landing/summary.md` claims, also has no row: no test reaps a root that does not exist with a process listed.
The code-tests lens raised the fallback gap and the assistant raised both; they are merged here.

## Holistic review

The diff makes the change the Goal asks for, by its "compare against both the link path and its target" option.
`rootProcesses` builds the root list once and calls the unchanged `namesRoot` for each form, which keeps the comparison in one function.
Focus verdicts:

- Shared link target: the resolved root keeps the `harden-sandbox` suffix, and `namesRoot` requires a separator or end of text after it, so a sibling under the same `.lym` target is not matched. Not a defect.
- EvalSymlinks failure: the error is dropped and only the written root is compared, which is the merge base's behaviour; a missing root has no process whose cwd lies under it. Not a defect; its coverage is F1.
- The new subtest's orphan names the root only through a link-resolved `Cwd`, with no `Exe` or `Argv`, so it fails on the merge base. Its input is built with `EvalSymlinks`, as the kernel would report it, and its expected value is the literal `[]int{12}`. Not a tautology.
- `TestSandboxVerbs`'s clear subtest starts the orphan with `Dir = root` and no argument, so on Linux only the resolved-root match can find it; its assertions on exit and the reported pid are unchanged.
- The `Root-Cause:` trailer on 19d52874 names the comparison of `/proc`'s resolved cwd against the unresolved root, which the diff removes.
- The `doc.go` lines and the `rootProcesses` doc comment match the behaviour as the diff leaves it.

The Goal names a `_lym` link; the sandbox root sits under `.lym` (`internal/hardenshed/sandbox.go:25`), and the diff targets that link, which is the Goal's intent.
The merge-danger call of two-way with a hardenshed blast radius holds: no record format, config meaning or outward push changes.
The test creates its link through `fslink.CreateDirLink` and names `.lym` through `lymdirs.DotLymDirName`, per `PATTERN-lym-files`.

## Rejected

- code-tests LOW at `internal/hardenshed/reap_test.go:129`, no negative row for the resolved root's boundary: the separator boundary lives in the unchanged `namesRoot` and is already pinned by the `root + "2"` row of `EndsRecordedProcessesThroughEndAndUnrecordedOnesThroughEndProcess`; the resolved root goes through the same function, so a second boundary row would duplicate that coverage.
