```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the proposals the fixer carried out for F19, F20 and F21.

**[MEDIUM] F19 (a): the kept tip is taken only before the first merge-in, so it misses the merge fixer's later commits on the task branch.**
- Changed text (D5): "Before Finalize first merges the parent branch into the task branch, it writes the task's own tip into the landing record, and a resume that finds that tip there keeps it", and "every search below runs over this kept tip".
- The case:
  1. Finalize merges the parent in. The `landing` gate fails.
  2. `gateAndFix` spawns the merge fixer in finalize mode. It commits on the task branch with the parent lock freed (`internal/landingshed/finalize.go:922-949`). Finalize then merges and lands again.
  3. The task lands by squash, which is the default. Examples are a hand squash after a Stuck push, or an adopter-first squash.
  4. The squash fallback asks `Fabric.HoldsWork` whether a commit holds the kept tip's work. That tip is the work as it was before the fixer. If the fixer rewrote those same lines, no first-parent commit holds the old work, and the row gets no landing commit.
- The rule (d) "no net code change" check has the same flaw. Say the kept tip adds nothing against the merge-base and the fixer's commits add the real change. The row then says "no net code change" for a task that landed code.
- Ancestry-first is not affected, because the later tip still descends from the kept tip. The squash paths are.
- A sounder change: refresh the kept tip before every merge-in, except when the previous merge-in fast-forwarded. Equivalently, record the task's tip after each non-fast-forward merge-in and after each fixer commit, never after a fast-forward. That keeps the review's hand-merge case safe and covers the fixer's work.

**[LOW] F21: the exclude line's pattern form is not pinned, and one exclude file serves linked worktrees anchored in different places.**
- Changed text (D1): "The records repository's `.git/info/exclude` holds an ignore line for the kept directory's `.tmp-*` names", and "one seeding covers every linked records worktree".
- `.git/info/exclude` patterns resolve against each worktree's root. The kept directory sits under the run's `_lym/`, which may lie below an anchor subdirectory: `junction.go:106-111` says AnchorRel is not always ".".
- A root-anchored line `_lym/<kept>/.tmp-*` would miss a worktree whose anchor is a subdirectory. The line needs to be unanchored, for example `**/_lym/<kept>/.tmp-*` or `<kept>/.tmp-*` with no leading slash. Otherwise the D1 `.tmp-*` criterion fails on a non-root anchor.

**F20:** I found no case against the proposal. It covers both a record commit that is an ancestor after another run lands on top, and a reset past a discarded local merge.

Tool calls:
1. Grep of `Finalize.mergeAndGate`/`gateAndFix` in `internal/landingshed/finalize.go`: my context lacked whether the merge fixer commits on the task branch after Finalize's merge-in.
