# hardenshed: reap orphans whose cwd sits under the root's link target

`Reap` in `internal/hardenshed` now ends an orphan whose working directory is under the sandbox root's link target.
Before, such a process escaped the reap when its command line did not name the root.

The root sits under a worktree's `.lym` link, and `/proc/<pid>/cwd` reports the link's target.
The comparison against the root as written never matched.
`rootProcesses` in `reap.go` now matches a process against the root as written and against the root with its links resolved by `filepath.EvalSymlinks`.
When the root does not resolve, or resolves to itself, only the written root is used.

Closes #164

## Design decisions

- The root is matched in both forms rather than replaced by its resolved form.
  A process that names the written path on its command line is still reaped, and so is one whose cwd the kernel reports resolved.
- A failed `EvalSymlinks` is not an error.
  Reap falls back to the written root, so a root that is already gone is handled as before.
- `namesRoot` is unchanged; `rootProcesses` calls it once per root form.
- `doc.go` states that a process names the root through its working directory, executable or command line, in both forms.
- The orphan-kill case in `TestSandboxVerbs` (`internal/hardencli`) no longer names the root on the command line.
  It starts the orphan with its working directory set to the root, which is the case that used to escape.

## Tests

`TestReap` in `reap_test.go` gains two cases:

- A root under a `.lym` link: one orphan has its cwd under the link target, another names the written root on its command line, and a third is unrelated.
  Only the first two are ended.
- A root that does not resolve: an orphan with its cwd under the written root is ended, and an unrelated one is not.

## To check by hand

- Root resolution has two outcomes the unit tests pin: the written root when it does not resolve, and both forms when it does.
  Confirm the two new `TestReap` cases cover both, since a review flagged these outcomes as untested.
- `TestSandboxVerbs` runs the Linux orphan case only on Linux; it was not run on another platform here.

## Merge danger

**Door:** two-way
**Blast radius:** hardenshed
