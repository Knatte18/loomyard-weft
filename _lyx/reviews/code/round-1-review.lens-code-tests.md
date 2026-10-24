```yaml
lens: code-tests
outcome: reported
```

**Per behaviour**
- Existing mutating-git matrix verdicts are kept. The rows of `TestFirstMutatingGitCommand` (`cluster_test.go:249-360`) now run through `firstMutatingGitCommand([]string{cmd})`. The rows read across the diff are unchanged, and every rubric row has a bound case in `TestFirstMutatingGitCommandScopesToForkRepositories`.
- Issue shape, `git init` followed by `-C` commit, `worktree add` and `worktree repair`, all in one command: pinned by "scratch repository experiment in one command".
- A repository created by an earlier command, then reached by `-C`: pinned by "repository created by an earlier command". It also exercises the `-b` valued option of `git init`.
- `cd <dir> && git commit` after `git init <dir>`: pinned by "cd into the created repository".
- `git init` inside a `cd`: pinned by "git init inside a cd".
- Subdirectory of a created repository: pinned by "subdirectory of the created repository".
- Relative `-C` under an anchored `cd`, including `..` cleaning: pinned by "relative -C under an anchored cd".
- Commit in a worktree added from a created repository: pinned by "worktree added from the created repository".
- `worktree add` from an unrecorded repository is not recorded: pinned by "worktree added from an uncreated repository".
- Commit outside every created repository: pinned by "commit outside every created repository".
- Prefix sibling: pinned by "sibling sharing the created name prefix".
- Unknown starting directory: pinned by "commit in the starting directory".
- `git push` from a created repository: pinned by "push from the created repository".
- `--git-dir=` form and `--work-tree` as a separate word: pinned by the two `--git-dir` / `--work-tree` rows. The separate-word `--git-dir` form is not pinned on the new path. It goes through the same `gitRepositoryOptions` branch as the `--work-tree` separate-word row.
- Bare `cd`: pinned by "bare cd after creating a repository".
- `cd -`: pinned by "cd - after creating a repository". It is a separate branch from bare `cd`, so it is not a duplicate.
- `git init` of the starting directory: pinned by "git init of the starting directory".
- `git init ..`: pinned by "git init of an ancestor".
- `..` escaping a `~`-anchored repository: pinned by "relative path escaping a created ~ repository".
- A `cd` in an earlier command does not carry over: pinned by "cd in an earlier command does not carry over".
- `$` anchoring: pinned by "scratch repository experiment in one command" and the `$S` row.
- `~` anchoring: pinned by the `~` row.
- Help on `init` or `worktree`: pinned only generically, by the existing help rows. `gitInvocationMutates` checks help before recording.

**Findings**
1. LOW, scope: `firstOperand` has no pinned behaviour for the `--` terminator. Location: `cluster.go` `firstOperand`. Evidence: no row has `--` in a `git init` or `git worktree add` command. A regression returning `""`/skipping it would go unnoticed. Add a row such as `git init -- /tmp/r` followed by `git -C /tmp/r commit`.
2. LOW, scope: valued-option skipping for `worktree add` (`-b`, `-B`, `--reason`) and for `init` (`--template` and the others) has no row. Location: `cluster.go` `gitWorktreeAddValuedOptions` and `gitInitValuedOptions`. Evidence: the only valued option exercised is `init -b` ("repository created by an earlier command", `cluster_test.go`). Add a row such as `git -C /tmp/r worktree add -b br /tmp/w` followed by `git -C /tmp/w commit`. Without the skip, `br` would be recorded as the worktree and the commit would fail the round.
3. NIT, scope: no row for a trailing slash on the created path (`git init /tmp/r/`, then `-C /tmp/r`). Also no row for an anchored `cd` restoring a known directory after a bare `cd` (`cd && cd /tmp/r && git commit`). Location: `cluster.go` `record` and `runsMutatingGit` (the `isAnchoredShellPath` branch). Both are behaviours the diff adds and no row covers.

**Duplicates under test economy:** none found. Every new row hits a distinct branch or acceptance criterion.

**Tool calls**
1. `sed -n 246,352p internal/burlerengine/cluster_test.go`: the context showed only the diff hunk of `TestFirstMutatingGitCommand`, not the existing rows.
