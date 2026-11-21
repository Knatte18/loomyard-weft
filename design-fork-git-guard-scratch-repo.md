# Burler-fix halts failed when a fork runs git in a scratch repo under .scratch

`fork-git-guard-scratch-repo` · Note · bug, burler, shuttle

Imported from [issue #177](https://github.com/Knatte18/loomyard/issues/177).

What happened: loom halted `failed` at Plan-Burler-Fix (anomaly `producer-hard-failure`, round 1 attempt 1).
The reason: the twill fixer fork ran a git-mutating command and the guard refused it with "a fork must never run a state-mutating git command".
The command created a throwaway probe directory under the worktree's `.scratch/`, ran `git init` for a bare repo and a work repo there, committed to the work repo, and pushed to the bare repo with `--force-with-lease`.
It never touched the task repository's own refs or working tree.
The fork was probing git push behaviour for a finding.

Why abnormal: the guard treats any `git init`, `commit` or `push` as mutating, including against a repo the fork created inside its own scratch directory.
The way forward it names ("inspect the target's working-tree changes, revert what the fork wrote") has nothing to revert here, so a harmless probe ends a run as hard failure.

What to do: scope the mutating-git check to the target repository and its remotes.
Allow git commands whose repo is a directory under `.scratch/`, or, if forks must not run them at all, say so in the fork prompt and give forks a sanctioned way to probe git behaviour.
Also make the halt message say when no target change was found.

Provenance: task single-repo-namespace-layout; notes loom-halt.md.