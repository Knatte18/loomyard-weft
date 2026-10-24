# Support log — fork git-mutation guard ignores repositories the fork created

## Interview

A darn run holds no interview: the scope comes from the board entry and a reading of `internal/burlerengine/cluster.go` and its callers.

## Rejected alternatives

- Report the violation as a recoverable stuck state when the target is unchanged: needs a before-and-after snapshot of the target's refs and working tree in the fix path, a larger change than the classification fix, and keeps a halt for a harmless experiment.
- Excuse every path outside the target worktree: the transcript carries no target path, and a worktree of the target placed outside it shares the target's refs, so location alone does not bound the risk.
- Carry the fork's cwd from the transcript into `shuttleengine.ForkReport`: widens the provider seam for a precision the lexical scheme does not need.
- Record `git clone` destinations too: not asked by the issue, and the destination default derives from the URL, more parsing for an unreported case.

## Review rounds

_No rounds yet._

## Question ledger

- Which of the board entry's two remedies to take? Self-pick: scope awareness, since it removes the false failure instead of softening it.
- Should a commit in a repository created under the target worktree, such as `.scratch/repo`, pass? Self-pick: yes, a nested repository holds no target state.
- Should `git push` from a fork-created repository pass? Self-pick: no, a push writes to another repository, possibly the target.
