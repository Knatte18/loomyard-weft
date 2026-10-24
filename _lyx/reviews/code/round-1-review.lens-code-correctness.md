```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced the changed code in `internal/burlerengine/cluster.go` with concrete inputs.

**Per function, case run, verdict**
- `firstMutatingGitCommand`: `[]` returns `("", false)`. The issue's command shape (`git init "$S/repo"` then `-C` commit, `worktree add`, `worktree repair`) returns `("", false)`. Correct.
- `forkRepositories.runsMutatingGit`:
  - `cd x` gives dir `x`, bare `cd` and `cd -` give an unknown dir, and a later `cd /abs` makes it known again. Correct.
  - A `cd` in an earlier command does not carry over, because `dir` is local to the call. Correct.
  - `cd -P /tmp/r` resolves to a bogus relative dir, which errs toward flagging. Correct.
  - `(cd x && git commit)` is not tracked, which is the documented open risk.
- `gitInvocationMutates`:
  - `init`: bare `git init` and `git init ""` give `.`, which is not recorded. `-b main .scratch/repo` records `.scratch/repo`. `git init -h` records nothing.
  - `worktree add`: recorded only when the repo is already contained, and the path is resolved against the `-C` dir. Correct.
  - `tag`: list-only is not flagged, write is checked against repos.
  - `push` is always flagged, and an unknown dir is never contained, so it is flagged. All correct.
- `record`: `.` and `..` are not recorded, `x/..` cleans to `.`, and `git init /tmp/r/` cleans to `/tmp/r`. Correct, apart from F3 below.
- `contain`: the sibling `/tmp/r2` does not match `/tmp/r`, trailing slashes are cleaned, and `/tmp/r/../../target` is not contained. Correct.
- `parseGitInvocation`: repeated `-C` is chained, a trailing `-C` is skipped, `--git-dir=x` and `--git-dir x` both clear `dirKnown`, a `-c` value is skipped, and `-h` returns early. Correct.
- `firstOperand`: `--` returns the next word, a valued option skips its value, and no operand returns `("", false)`. Correct.
- `isAnchoredShellPath`: correct.
- `resolveShellPath`:
  - Absolute paths, a relative path under a relative dir, `~/` and `$S/` all resolve correctly.
  - A `..` under a `~` or `$` anchor is clamped at the anchor, which is wrong (F1).
- `forkViolation`: the message and the returned command are unchanged, and the first offending command is reported. Correct.

**Findings**

- F1, LOW, design, `cluster.go` `resolveShellPath` (the `anchor+path.Clean("/"+rest)` branch, lines around 243–247): a `..` under a `~` or `$` anchor is clamped at the anchor, so it resolves inside the anchor instead of above it.
  - Run: `git init "$S"`, then `git -C "$S/../target" commit -m x`. The path `$S/../target` cleans to `$S/target`, which is contained in the recorded `$S`, so the commit is excused.
  - `git -C "$S/.." commit` resolves to `$S` and is excused the same way. `git init ~` followed by `git -C ~/.. commit` behaves the same.
  - This contradicts the decision record's rule that an unresolvable directory never matches a recorded repository.
  - The existing row "relative path escaping a created ~ repository" only covers a root of `~/r`, so it does not catch this.
  - Fix: when `..` would climb above the anchor, return an unmatchable result instead of clamping.

- F2, LOW, design, `cluster.go` `parseGitInvocation` (the `gitRepositoryOptions` case): the bounds exclude `--git-dir` and `--work-tree` only as options. The same redirection by environment or config is not excused-against.
  - Run: `export GIT_DIR=/target/.git`, or `git -C /tmp/r -c core.worktree=/target commit`. Both leave `dirKnown` true, so the commit inside the recorded repo is excused.
  - The record's stated purpose for the bounds is to close the routes by which a mutation inside a recorded directory could reach the target's repository.
  - Fix: treat `-c core.worktree=` and `-c core.bare=` as repository options in `parseGitInvocation`.
  - An env-var assignment such as `GIT_DIR=` or `GIT_WORK_TREE=` in an earlier segment of a fork's commands also redirects it. Making the audit account for that is a decision for the author.

- F3, LOW, design, `cluster.go` `record`: the "starting directory or ancestor" guard only rejects paths made entirely of `.` and `..` elements.
  - Run: `git init ../<startdir-basename>` is recorded as the relative root `../<name>`, although it names the starting directory itself.
  - Later `git -C ../<name> commit` is then excused.
  - The fork would have to know the target directory's name, so this is an edge case.
  - Fix: either state it as a documented open risk, or add a row and refuse to record any relative path that begins with `..`.

**Tool calls:** none. The diff and `cluster.go` were already in context.
