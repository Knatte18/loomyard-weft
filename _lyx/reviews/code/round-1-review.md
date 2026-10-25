---
verdict: APPROVED
---

The diff makes the change the amended board entry asks for.
`drivePasses` drops every done run that has no pass in flight before `d.runs` is set, so `recordOwed`, pass planning and `runsInFlight` all read the filtered list.
A done run's `Runs` entry leaves `state.json` through the existing unlisted-run drop, including on a tick where a dobby start owed every run a full pass.
A done run with a pass in flight keeps its slot until the exit is read, and `collectExits` runs before the filter, so the slot and the entry go on the same tick.
The board tidy's landing count and the beamcli listing are untouched.
The `doc.go` bullets and the `startReady` comment match the new behavior, and the two new tier0 subtests pin the start beside done records, the empty state record and the in-flight slot.

## Rejected

- Lens code-correctness, LOW: `wasCovered` and `childSeen` memory stays for a run dropped as done. The same leftover exists for any run that goes unlisted, the effect is at most one extra pass that self-heals, and the diff adds no new path to it.
- Lens code-tests, LOW: the in-flight subtest does not assert the done run's entry is present before the exit. Keeping the entry is existing behavior the diff does not change, and the acceptance criterion is the slot, which the subtest pins.
- Lens code-tests, NIT: the new subtests sit in `TestStartReady_RefusalsAndBackoff`. The decision record allows a subtest in the existing dobbyengine tests, and no test is duplicated.
