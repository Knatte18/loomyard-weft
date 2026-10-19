# Case brief: Discussion-Burler-Review refused on a read-only git command

## 1. The question the next judge must answer

Is the burler fork's git command a real state-mutating git call, or does the git-mutation guard in `internal/burlerengine` misclassify a read-only command?
If it is a misclassification, the guard needs a code fix before the run can resume, and that fix is outside the tackler's two-file write scope.

## 2. The facts

- The stop is `kind: producer` with `transient` empty, first producer `Preflight`, failed at `Discussion-Burler-Review`, round 1 attempt 1.
  Source: `_lyx/shed/ubuntu-2404-support/drive-reports/drive-envelope-20261019-170838-153e.json`, field `error`.
- The refused command is `git help -m cherry-pick 2>/dev/null | col -b | grep -n -A14 "^ *--empty" | head -30`.
  Source: `.lyx/loom/driver.log`, line for `2026-10-19T17:08:38.972` (`shed: step refused`), and `.lyx/shed/ubuntu-2404-support/steps/a42b40cb49e84e85.trace.log`, same line.
- The fork is `agent-a2dfed49a3f9fd78e.jsonl` (lens "interactions"), one of four review forks in round 1.
  The review's round-1 fork set ran from 16:59:57 to 17:08:18.
  Source: `.lyx/loom/driver.log`, lines at 16:59:57, 17:06:19 and 17:08:18.
- The guard's regex is `mutatingGitPattern` in `internal/burlerengine/cluster.go`, lines 24-26.
  It matches `git`, then any run of non-operator tokens, then a whole token from a list that includes `cherry-pick`, `checkout`, `branch`, `apply`, `tag` and others.
  It does not read the git subcommand position, so a token after `help`, `log` or any other read-only subcommand still matches.
- The refusal is raised at `internal/burlerengine/cluster.go`, lines 58-63, in `forkViolation`, which loops over `fork.BashCommands` and returns the error on the first match.
- The command as written is read-only.
  `git help -m cherry-pick` shows a man page: `-m` is `--man`, and `help` does not touch refs, the index or the worktree.
  The pipeline to `col`, `grep` and `head` only formats output.
- Attempt count: this stop has no prior resume.
  The envelope's `history_length` is 4 and `status_moved` is false.
- The stderr log, `.lyx/shed/ubuntu-2404-support/steps/a42b40cb49e84e85.stderr.log`, holds only the refusal line.

## 3. What I ruled out, and how

- A real mutation by the fork. Read the command string in the envelope, the trace and the driver log: no ref, index, worktree or HEAD change is in it, and none of the `git` subcommands in the list is actually invoked.
- A git state problem in the task worktree. Nothing in the logs or the envelope names one, and the refused command is the only git call the stop names.
  I did not inspect the worktree's git state directly.
- A transient error. `transient` is empty, and the failure is a logic refusal, not a timeout or an environment fault.
- A cause in the driver's rules. The driver's stop report says `no rule settles the stop: the tackler decides`.
- A plain re-launch settling it. A re-launch runs the same review with the same guard. The fork could issue the same `git help` line again, so a resume would most likely hit the same refusal.
  That is a guess from the code, not a run result, so this is a judgment and not a proof.

## 4. Hypothesis (unverified)

Hypothesis: the guard's regex treats any `git ... <token>` as a mutating call when the token is one of its list words, so a read-only `git help -m cherry-pick` trips it.
The fix is in code the tackler cannot edit: tighten `mutatingGitPattern` to match only the subcommand position, or allow the read-only `help` subcommand, and add a test case for `git help -m cherry-pick`.
A stronger agent or the run's parent should make that change, under the usual tests and the gate.
Until it lands, the fork may keep emitting read-only git commands that trip the guard.
