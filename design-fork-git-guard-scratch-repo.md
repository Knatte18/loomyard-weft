# Fork git-mutation guard fails a run on git commands confined to a scratch repo

`fork-git-guard-scratch-repo` · Task · bug, shed, burler · running · PR-Review

Imported from [issue #843](https://github.com/Knatte18/loomyard/issues/843).

**What happened**

A loom run halted `failed` at `Plan-Burler-Fix` (anomaly `producer-hard-failure`).
The shedadapters check reported that the twill fixer fork "ran a git-mutating command" and refused with: "a fork must never run a state-mutating git command".
The flagged command was a single compound shell line.
It created a throwaway repository under the session's scratchpad directory (`git init`, an empty commit, `git worktree add`, `git worktree repair`) to test git behaviour, and never touched the task's own worktree or repository.

**Why it is abnormal**

The guard's intent is to stop a fork from changing the target's git state.
Here every git invocation was aimed at a repository the fork had just created in its own scratchpad, so no target state was at risk.
The guard appears to match on the git verbs alone without checking which repository they act on, so a harmless experiment fails the whole round and halts the run.
The advised recovery ("inspect the target's working-tree changes, revert what the fork wrote") has nothing to revert in this case.

**What to do**

Make the fork git-mutation check scope-aware: ignore git commands whose repository (`-C <path>`, `cd <path>`, or `git init <path>`) resolves inside the session scratchpad or another path outside the target worktree.
Alternatively, report the violation as a recoverable stuck state rather than a hard failure when the target's working tree and refs are unchanged after the fork.

Provenance: task `orch-strand-cycle`, note `loom-halt.md`.
