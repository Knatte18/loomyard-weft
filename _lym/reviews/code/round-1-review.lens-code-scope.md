```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff back to a goal point of the decision record.

**Decisions** (the `## Goal` board entry; the record has no `Added after Discussion` entries):
- **G1:** Resolve symlinks on the root in `namesRoot`, or compare against both the link path and its target.
- **G2:** Add a test whose orphan has the link working directory and no root on its command line.

**Per change:**
- **C1** (`internal/hardenshed/reap.go`, `rootProcesses`): builds `roots` from `deps.Root` plus its `filepath.EvalSymlinks` form when that differs, and matches a process against either through `namesRoot`. Builds G1 by its "compare against both" option. The comparison stays in `namesRoot`; only the root list is built one level up in its sole caller, so the change the goal asks for is made.
- **C2** (`internal/hardenshed/reap.go`, `rootProcesses` doc comment): forced by C1, which changed the function's contract.
- **C3** (`internal/hardenshed/doc.go`, two added lines on root matching): forced by C1, since the package doc describes which processes Reap ends.
- **C4** (`internal/hardenshed/reap_test.go`, subtest `EndsAnOrphanWhoseCwdIsUnderTheRootsLinkTarget` plus the `fslink` and `lymdirs` imports): builds G2. Its orphan has only `Cwd` set, to the resolved path, with no `Argv` and no `Exe`.
- **C5** (`internal/hardencli/cli_integration_test.go`, orphan started with `Dir = root` and no root argument): builds G2 with a real process. It also undoes the workaround the goal's "Why it is abnormal" names, where the test was changed to put the root on the command line.

**Per decision:**
- G1: built by C1, with C2 and C3 forced.
- G2: built by C4 and C5.

The goal says `_lym` link, but the sandbox root sits under the worktree's `.lym` link (`SandboxRoot` joins `lymdirs.DotLymDirName`, `internal/hardenshed/sandbox.go:25`). The diff targets the link the root actually sits under, which matches the goal's intent, so this is no finding.

No `Pattern-Departure:` trailer appears in `git log 13cebe3d..HEAD`, and none is needed.

**Findings:** none. Every hunk traces to G1 or G2 or is forced by C1, and both goal points are built.

**Tool calls:** none; the inherited context held the diff pieces, the decision record, the commit log and `SandboxRoot`.
