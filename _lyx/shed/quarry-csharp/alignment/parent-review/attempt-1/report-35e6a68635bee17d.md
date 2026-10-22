---
verdict: APPROVED
---

## Scope against the board entry

The entry `quarry-csharp` asks for three things, and the decision record covers each:

- Raise quarry in `go.mod` and fix the zero-`Lang` library break: D1.
- Pick the alphabet per file, by extension, wherever LoomYard asks quarry for glyphs: D2, D3 and D4.
  The record's Scope states its reading of "per file": a C# member glyph names a namespace and carries no file, so the plan names its alphabet once through `language:`, the planner picks it from the extension of the files it plans, and D4 checks that pick against the plan's path targets.
  Path entries use the Go unit alphabet because quarry spells a path only there.
  This is the entry's extension rule applied to quarry `v0.4.0`'s alphabet facts, declared rather than silent, so it is no departure from the entry's direction.
- The planner, Plan-Review and the card kick-start get C# glyphs in a C# hub: D7, D8, with D5 and D6 as the parts those consumers need to work on C# answers.
  The kick-start criterion holds only while the pack exists, so it does not contradict `kickstart-pack-remove`.

Nothing in the record belongs to another entry:

- Enclose wiring stays with `enclose-checks`; this run only takes in `v0.3.0` through the version raise, which that entry names.
- .NET verify gates (`dotnet-scoped-verify`, `dotnet-verify-protocol`), language lens sets (`lenses-language-sets`) and the pack's removal (`kickstart-pack-remove`) are named out of scope.
- C# counterparts of the Go-toolchain checks are out; D5 only skips the Go checks under `csharp`, which a C# hub needs before the planner and Plan-Review can use C# glyphs.

`tsa-quarry-audit` states the old pin in an audit note; no decision there depends on it.

## Overlaps with other runs

- `research-in-discussion` (row 0, Darn-Write): both runs change `contracts/stencils/loom/loom-rubric-plan-review.md`.
  That run adds a new `## Also flag` bullet on research steps in cards; this run (D8) rewords the existing bullet on member glyphs for an `Edit` card so it covers Go and C# file paths.
  Different places in the same file, no conflicting behaviour: a text-only overlap.
  Landing order: unknown, since the peer's row is 0 against this run's `own_row` 3.
- `darn-verify-fixes` (row 5): its decision record names `internal/planglyph` only as an example of a diff's impacted test set; it changes nothing this run changes.
  No overlap.
- `machine-handover`, `shuttle-open-agent-defaults`, `decision-add-notifies`: no shared files or behaviour.

No parent commits are listed.
