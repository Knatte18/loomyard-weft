# beam merge-parent: the merge task merges and lands with no pull request

The merge task `lym beam merge-parent` files failed three ways, and this change fixes each.
Its writer was refused the verbs its task names, it landed as a squash that dropped the parent's commit, and it took a pull request nobody needed.

## The writer may merge

The sandboxed-role guard now lets a merge task's darn writer run `lym fabric merge-in`, `merge-stage`, `merge --continue` and `merge --abort` in its own worktree.
`merge <branch>` stays refused, and so does every other role of the task.
The guard reads only the strand name, so `darnengine.IsMergeTaskWriter` judges the role and the slug's `-merge-in` suffix.
The suffix moved from `beamcli` to `darnengine`, which owns the writer's role.

## The landing keeps the merge

`pull_request` on a board entry takes a new value, `merge-in`, which `lym beam merge-parent` sets when it files the task.
`ResolvePullRequest` answers no for it under every parent mode and GitHub rule, without reading the rules, and marks the decision `TrueMerge`.
Finalize then runs the parent-side merge without a squash, whatever `landing.yaml`'s `squash` says, so the beam gets a fast-forward or a merge commit.
The parent's commit stays an ancestor of the beam, and the beam's pull request into its parent no longer reports the conflicts the merge settled.

## Design decisions

- The mark is a value of the entry's `pull_request` field, not a run setting, since `PATTERN-landing` lets only the resolver decide this and bars a run setting from carrying it.
- The PR-Review row stays in the darn recipe and passes at once on the resolver's no, opening nothing, because recipes have no per-entry rows.
- Describe still runs when the Code-Review perch is on, since the issue's wanted behaviour asks only for the pull request and PR-Review to go.
- A fast-forward landing keeps the task's commits and writes no landing commit, so the change description reaches no commit message on that path.
- A beam branch GitHub protects refuses the direct push, and the run stops Stuck.
  Its way forward leaves out `pull_request=merge`, which the `merge-in` field would ignore.
- A slug ending in `-merge-in` on a task `lym beam merge-parent` did not file also gets the merge verbs; the guard stops mistaken calls, not hostile ones.

`pattern/PATTERN-landing.md` records the new rule.
Tests pin the guard's verbs and roles, the resolver under every mode and rule, the unsquashed landing with no rule read, the protected-push text and the filed field.
