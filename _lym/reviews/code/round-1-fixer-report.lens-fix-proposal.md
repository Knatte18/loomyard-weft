```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals (F1–F5) that the fixer carried out as proposed.

I found no case where a proposal makes behaviour wrong. Two wording issues came out of the proposals, both LOW:

1. **LOW**, F3, the Warn in `internal/loomcli/sharedbootstrap.go`: "the next start or resume will replace it" overstates what resume does.
   - `resumeFromRunState` runs the restore only when it spawns a Go driver or wakes a parked one.
   - A refused resume, a done run, or a running run with a live driver returns without calling `restoreStatusPaneBeforeWake`, so the strand is not replaced.
   - `start` does restore on each of its entries into the bootstrap. A wording such as "the next restore will replace it" would hold.

2. **LOW**, F4, the Design decisions bullet in `_lym/landing/summary.md`: the stated exception covers only a successful replace.
   - `ReplaceStrand` selects the new window only when `addErr == nil` (`reedengine/replace.go`, `endedCurrent && replacingWithOverview`).
   - On a failed add, the old window is already killed, and `killStrandPanes` was told not to move the view to the strands window (`!replacingWithOverview` is false). A viewer who sat on that window therefore lands wherever tmux puts them.
   - "Leaves the viewer's window as it is, except … moves the view to the replacement's window" does not describe that path. I did not verify which window tmux picks.

The other proposals held up:
- **F1:** `start` and the `wiring.go` closure are the only callers of `ensureStatusStrand`, and both treat the sentinel as non-fatal. After a failed replace, the next `start` finds the strand missing (add) or stale (replace), so the `status_pane` way forward still applies.
- **F2:** the subtest already depended on the earlier subtests for its session, so requiring the strand they leave adds no new ordering dependency.
- **F5:** no breakage found.

Tool calls: none.
