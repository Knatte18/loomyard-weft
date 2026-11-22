```yaml
lens: fix-claims
outcome: reported
```

Task: check every claim the fix to card 14 made against HEAD.

**[MEDIUM]** Card 14, `prReviewGotoAdmitted` bullet: "`StopStrand` does nothing for a strand reed does not track, so the seam needs no separate liveness read."

- **Summary:** This sentence, taken from the review's proposal, is only half true. `StopStrand` removes nothing reed does not track, but it can still write the run record.
- **Evidence:** In `internal/shuttleengine/stop.go:150-158`, `StopStrand` calls `FindRun` first. When a run record exists, `stopRecorded` (`stop.go:167-184`) moves a record that still reads running, and whose output files are not all present, to `stopped`. It also calls `reportStoppedSession`, which reports a session end to the usage hook. It then calls `removeStrandIfPresent`, which is the only step that skips an untracked strand (`stop.go:226-234`).
- **The liveness read was not equivalent either.** The old read passed only `Live && !Retiring`. `strandRemovable` (`stop.go:216-218`) also removes a held strand and an own-window strand with a dead pane, and it does not skip a retiring one. So "needs no separate liveness read" is a behaviour change, not a redundancy, and the card should say so.
- **Suggested change:** Say that `StopStrand` records the stop on a run still reading running and removes only a strand reed tracks as removable. Keep the conclusion only if that wider stop is acceptable for a run moved back onto PR-Review.

The other claims hold:
- `goto` is lightweight-wired: `cli.go:325`, and `arm.go:267-268`.
- `wireLightweight` leaves `c.reed` and `c.runner` unset: `wiring.go:411-452` has no assignment to either.
- `stepStopRunner` returns `c.runner` when it is set and otherwise builds the runner from the reed and shuttle config: `looparm.go:175-198`.

Tool calls:
- One Bash call read `stop.go:175-225`, the `wireLightweight` tail in `wiring.go`, and the `armAt` lines in `arm.go`. My context showed `stopRecorded` only up to line 175, `wireLightweight` only up to line 441, and none of `arm.go`.
