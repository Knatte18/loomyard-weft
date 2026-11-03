```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3 as the fixer carried them out. I made no tool calls.

**F1 (zero-exit orphan row): no findings.**
- Without the nil mapping, `Wait` returns `exec.ErrWaitDelay`. Per `build.go:48`, `exitCodeOf` would turn that into a non-zero code, so the ZeroExit row fails on exit code, outcome and `ExitCode`.
- The row does reach the branch. `killLeftovers` runs only after `Wait` returns, and `Wait` returns only after the 100 ms delay cuts the pipe the orphan holds.

**F2 (shared build orphan row): no findings.**
- If `runBuild` passed a zero delay, `Wait` would block on the orphan. `killLeftovers` runs after `Wait`, so nothing would end the orphan and `g.wait` would time out. The row fails with the defect in place.
- Splitting `helperOrphanEnv` into `tier` and `build` values keeps the tier scenario intact. The build row sets `build`, so the tier invocation never orphans, and tier0 is blocked by the failed build anyway.
- The build row only covers a failing build exit. The zero-exit nil mapping is not exercised through `runBuild`. Both callers share `runTiedChild`, and the tier ZeroExit row pins that branch, so this is not a gap.

**F3 (summary line 18): LOW**
- **Changed text:** `_lym/landing/summary.md`, the new line "Linux keeps a group id pinned while any member lives, and the sweep finds nothing once the group is empty."
- **Summary:** the sentence drops the residual window the review itself records. A new group leader that reuses the pid after the group empties would be swept.
- **Evidence:** the review's "Focus verdicts" entry for `killLeftovers` says "only a pid-space wrap inside a window of microseconds, up to the delay on the orphan path, could hit a new group leader with that id". The new text reads as an unconditional guarantee, which is milder than the `KillTree` start-time overclaim it replaces but still stronger than the code.
- The review's own fix text asked for this wording, so the fixer followed the proposal. Qualifying it with "short of a pid-space wrap" would make it exact.

Calls: none; the context already held every changed file and the review.
