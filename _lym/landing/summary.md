# Harden reap ends an orphan whose cwd is under the root's link target

Fixes [issue #164](https://github.com/Knatte18/loomyard/issues/164).

The harden sandbox root is `<worktree>/.lym/harden-sandbox`, and a worktree's `.lym` is a link into its weft directory.
`/proc/<pid>/cwd` reports a working directory with its links resolved,
so `namesRoot` in `internal/hardenshed/reap.go` never matched a process whose working directory was under the root and whose command line did not name the root.
Such a process escaped every reap.

## Changes

- `rootProcesses` in `internal/hardenshed/reap.go` resolves the root's links once with `filepath.EvalSymlinks` and counts a process as naming the root when it names either the root as written or the resolved root.
  When the root does not resolve, as when it does not exist yet, only the written root is compared, as before.
- `TestReap/EndsAnOrphanWhoseCwdIsUnderTheRootsLinkTarget` builds a worktree whose `.lym` is a link to another directory and reaps a process whose working directory is the resolved path, with no root on its command line.
  It fails on the merge base: `EndProcess` is never called.
- The `harden sandbox clear` integration test in `internal/hardencli/cli_integration_test.go` again starts its orphan with the root as its working directory and no root on its command line, instead of passing the root as the script's `$0`, which had hidden the gap.
- `internal/hardenshed/doc.go` states that the root matches both as written and with its links resolved.

## Merge danger

**Door:** two-way
**Blast radius:** hardenshed
