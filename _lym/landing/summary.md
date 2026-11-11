# Worktree-Create deletes a reused slug's path of git-ignored output

A build or language server can write into a pair's task worktree path after the pair was torn down.
It leaves a directory of `obj/` or `bin/` output that no worktree registers.
`fabricengine.Add` refused that directory with `*ErrTaskPathTaken`, so the next run of the same slug halted at Worktree-Create until someone inspected and deleted it by hand.

`Add` now deletes such a directory and makes the pair there.
It deletes the directory only when all of these hold:

- it is a real directory, not a file or a link;
- no worktree of the repository registers it;
- it holds no `.git` entry at any depth;
- `git check-ignore`, run in the acting worktree, ignores every file under it at the same repository-relative path.
  A tracked path never counts as ignored, and an empty directory passes.

Anything else is refused with `*ErrTaskPathTaken` as before, and batten's halt and way forward for that refusal are unchanged.
A `check-ignore` that fails proves nothing, so the path is kept and the failure logged.
The delete goes through `removeWorktreeLeftover`: contained by an `os.Root` under the hub, retried against a late writer, and recorded as `KindPathRemoved`.
A delete that still fails returns an error wrapping `ErrStrayPathKept`.

## Design decisions

- **The proof is the repository's own ignore rules, not a list of build directory names.**
  `PATTERN-code-conventions` forbids hard-coding a language, and `obj/` and `bin/` are one toolchain's names.
  Git's ignore rules already name what each repository treats as generated output.
- **The fix is at Worktree-Create, not Worktree-Teardown.**
  Teardown already deletes a stray directory refilled during the removal, through `RemoveStrayTaskPath`.
  In the reported case the output was written a day after teardown, so only the next Create can see it.
- **A new destruction-chokepoint exception.**
  The judge and the delete live in `destroy.go` as `removeIgnoredTaskPath`.
  The `PATTERN-fabric-boundary` background and the package doc's chokepoint section now list three exceptions in place of two.

## Risk

A directory at a slug's path that holds only ignored files is deleted even when those files were not build output: a git-ignored `.env`, for instance.
Teardown already deletes ignored files with the worktree, so this matches what lym does to ignored content at a pair's path.

## Tests

`TestAdd_TaskPathLeftover` (tier 1) covers four cases in one table:
a directory of ignored output is deleted and the pair is made;
an empty directory is deleted;
a directory with a file git does not ignore is refused and left on disk;
and a directory with a `.git` entry under an ignored directory is refused and left on disk.
