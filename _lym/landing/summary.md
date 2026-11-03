# Name the dirty paths when a landing fixer halts on a dirty tree

When a landing fixer session ended with the task worktree dirty and its attempts spent, the halt reason said only "ended without a usable turn (dirty)".
The operator could not tell which paths were left behind, or whether the branch needed work at all.
That is what happened in Parent-Align: the fixer reported `verdict: fixed`, the gate had passed, and the run still halted as `blocked` with no hint of the cause.

The halt reason for the dirty end now names the cause:

```
the fixer session of <row> left uncommitted or untracked paths in the task worktree after its last turn: <paths>; session log: ...; report: ...
```

Its way forward is "commit or remove them on the task branch, then `lym loom resume`", replacing the generic "fix or clean the task branch".
The timeout and no-report ends keep the generic wording.

Closes #6

## Design decisions

- The end-state classification is unchanged.
  The dirty end is still reached only when the tree holds uncommitted or untracked paths after the last turn and the fixer attempts are spent.
  A tree that is clean at that point is not classed dirty, so the paths in the new reason are the evidence for the halt.
  The issue's first suggestion, treating a passed gate plus a finished turn as a clean end, is not implemented: a tree with leftover paths is not clean, and a resume would carry them into the landing.
  The message change makes the real condition visible instead.
- `fixerOutcome` gains a `dirty` field holding the paths, set in `judge` where the dirty end is halted, and `haltReason` formats it with `listShown`.
- `internal/landingshed/doc.go` lists the dirty halt, with its named paths, among the halts that open with `LandingStopPrefix`.

## To check by hand

- The cause of the original halt is not established here.
  If the Parent-Align fixer's `verdict: fixed` run left a clean tree, a halt like the reported one would need a different explanation.
  Reproduce with a fixer that leaves a file behind and confirm the halt names it.
- `fixerrun_test.go` updates the dirty-end case to assert the paths and the new way forward, and switches an outcome comparison to `reflect.DeepEqual` because `fixerOutcome` now holds a slice.
  The tests were not run in this session; run `lym gate test ./internal/landingshed`.
