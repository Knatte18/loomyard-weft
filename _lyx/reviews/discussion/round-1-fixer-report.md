# Fixer report — discussion, round 1

## Changed

All in `_lyx/discussion/decision-record.md`.

- F1 — fixed as proposed.
  D1's first keep bullet keeps the position of a Go-filled marker relative to another marker, a heading, or the end of the body.
  D2 sends the two position rows (`{{.kickstart_pack}}` end of body, `{{.kickstart_pack_inline}}` before `## The sandbox rule`) to `markers_test.go` as one table of marker positions.
  Acceptance criterion 5 now says "moving a position-pinned marker" instead of "swapping two order-pinned markers", to match.
- F2 — fixed as proposed.
  D2 gains a bullet: the Go-value table test and the twill host-neutral scan each carry `//lyx:guard`, so a rename of `shuttleengine.MessageTail`, `shedrun.ParkMarkerFileName` or `gatecli.SlotBusyExit` outside `contracts/stencils` still fails its card gate.
  The Notes bullet on dropping the guard is deleted.
- F3 — fixed as proposed.
  D4's rationale drops Plan-Review from the bound and names stencil changes landed outside loom and darn as let through.
- F4 — fixed as proposed.
  Acceptance criterion 2 excepts sentences holding a marker, a Go-named value, a twill host word, a kept branch anchor, or text a PATTERN scan checks.
- F5 — fixed as proposed.
  D3's fifth bullet exempts stencil prose from `loom-rubric-merge-review.md`'s "a behavior of the interaction with no coverage" item, and acceptance criterion 8 names the merge-review rubric.
- F6 — fixed as proposed.
  The first Notes bullet says the literal key rows such as `class` and `first_card` go by D1, since `roundRecord` and `overviewFrontmatter` are unexported.

## Disputed

None.

## Deferred

None.
