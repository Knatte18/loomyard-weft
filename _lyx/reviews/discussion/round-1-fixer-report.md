# Round 1 fixer report — discussion

Every premise was checked against the code: the fork boilerplate's last line in `contracts/stencils/twill/twill-directive.md`, `configengine`'s in-memory fill (`internal/configengine/config.go:127`), `discussion_producer`'s refusal of an empty value (`internal/loomengine/config.go:726-730`), the 15-wait give-up and 300000 ms timeout in the stencil, `plan_timeout_min: 120` in `_lyx/config/loom.yaml`, and `burlerengine.ResolveFan`'s refusals.
All hold.
The review's line numbers are off against the current record; each finding was located by its quoted content.

## Changes

- F1 (fixed as proposed): D4's severity bullet states that the marker replaces only the parenthesised phrase, so Discussion keeps its bytes, and that every host value reads as a noun phrase in that slot.
  D5's severity rule is rewritten as such a phrase.
  "Report defects only, never a passed check" moves into D3's spawn point as a line the writer adds to every fork's prompt.
  A new acceptance criterion requires the Plan fork boilerplate's last line to carry the severity rule as one well-formed sentence.
- F2 (fixed as proposed): the refuses criterion gains a side-token member without a test prefix under a doc root such as `cmd/`, and a host-geometry identifier under `cmd/`.
  The passes criterion gains a host-geometry identifier outside `internal/` and `cmd/`, such as under `tools/`.
- F3 (fixed as proposed): D1's rationale says an absent key takes the template's `single` through `configengine`'s in-memory fill, and an explicit empty value reads as `single`, unlike `discussion_producer`, which refuses it.
  The decision and its criterion are unchanged.
- F4 (fixed as proposed): the Open risks entry is rewritten to the bounded form, and the same sentence is added to D2's Bound.
- F5 (fixed as proposed): D8 gains a Bound line naming the fan resolver's refusals and the forks' read-only status per D2.
- F6 (fixed as proposed): D5 and D8 each gain a sentence saying their texts are rendered as written, since the bench file is unreadable to the plan writer.
  D5's sentence also covers the fork-prompt line F1 added to D3.

The support log needed no change.

## Disputed

None.

## Deferred

None.
