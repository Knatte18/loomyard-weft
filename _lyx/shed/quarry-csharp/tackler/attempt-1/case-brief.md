1. The question the next judge must answer.

Is the refusal of the Plan-Burler-Fix round-2 fork a false positive of the git-mutation guard, and if so, does the guard need a code change before the run resumes, or does a plain re-launch settle it?

2. The facts, each with a pointer.

- The stop envelope `_lyx/shed/quarry-csharp/drive-reports/drive-envelope-20261022-093815-54f4.json` names the stop as `kind=producer`, `state=failed`, `transient=""`, at `Plan-Burler-Fix (burler-fix)`, round 2, attempt 2.
- The refused fork is `agent-af6ad1463741a3a48.jsonl`. Its one Bash command, as quoted in the error, is `Q=/home/knatte/Code/quarry-LYXHUB/quarry; sed -n 225,320p $Q/internal/engine/name.go; git -C $Q log -1 --format='%h %d' ; git -C $Q tag --points-at HEAD`.
- Of its four segments, only `git -C $Q tag --points-at HEAD` is a git invocation that the guard counts. Its subcommand word is `tag`. The `log` segment and the `sed` segment are read-only.
- `internal/burlerengine/cluster.go:24-28` (`mutatingGitSubcommands`) lists `tag` and `branch` as mutating with no regard to their arguments.
- `internal/burlerengine/cluster.go:35-52` (`commandRunsMutatingGit`) decides only on the subcommand word. The `--points-at` flag is not examined, so the listing form counts as a mutation.
- `internal/burlerengine/cluster.go:180` raises the hard error that stopped the run.
- `internal/burlerengine/cluster_test.go:273` pins `git tag v1.0` as mutating. No case pins the listing forms `git tag --points-at`, `git tag -l` or `git branch --list` either way.
- `internal/burlerengine/doc.go:293-294` says a Bash command is git-mutating when "the subcommand word ... mutates". The doc does not say whether `tag` and `branch` listings are exempt.
- The target repo `/home/knatte/Code/quarry-LYXHUB/quarry` is clean (`git status --short` empty). Its HEAD is `ca716c2` with tag `v0.4.0`, and `git tag --points-at HEAD` prints `v0.4.0`. Its stash list is empty. Nothing in the target shows a write from the fork.
- The driver log `.lyx/loom/driver.log` at 09:38:15 records the refusal (`shed: step refused`). The same log shows the round had already reached `burler: fix joined ... fixOutcome=done` and the fork's `Fix-check lens` siblings finished. The refusal came after the fix completed, in the post-fork audit.
- The shed driver's report `_lyx/shed/quarry-csharp/drive-reports/drive-report-20261022-093815-47d7.md` already records one arbiter attempt, resumed with verdict `resume`, at `_lyx/shed/quarry-csharp/arbiter/attempt-1/verdict.yaml`. That attempt is about an earlier stop, not this one.

3. What I ruled out, and how.

- The fork wrote files: ruled out by the clean `git status --short` in the target repo. The guard's own write count is not in the envelope, so this rests on the target's state, not on the fork transcript.
- The fork ran a state-changing git command: ruled out by reading the command. `git -C $Q tag --points-at HEAD` and `git -C $Q log -1` change nothing. The stop is a classifier decision, not a real mutation.
- A transient infrastructure fault: ruled out. `transient` is empty and the error is deterministic from the command text.
- A plain re-launch as the fix: not ruled out, and this is why I do not resume. The fork picks its own commands, so a relaunch may pass or may repeat `git tag --points-at HEAD` and stop again. I cannot tell which from the files I read.

4. My hypothesis (a hypothesis, not a finding).

The guard counts `tag` and `branch` as mutating whatever their arguments. That makes a read-only listing such as `git tag --points-at HEAD` a false positive, and the refusal is a defect in `commandRunsMutatingGit`, not in the fork's work. The fix belongs in code: the classifier needs a listing exemption for `tag` and `branch` (for example `--points-at`, `-l`, `--list`, `--contains`), with a test case beside `cluster_test.go:273`. That change is a judgment about what counts as read-only, so it belongs to a stronger agent or the run's parent. Until it lands, a relaunch that repeats the command will stop the same way.
