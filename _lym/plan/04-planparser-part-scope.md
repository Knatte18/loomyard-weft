# Card 4 — planparser-part-scope

**Create:**
- `internal/planparser#PartScope`
- `internal/planparser#checkPartSpotTarget`
- `internal/planparser_test#TestValidate_PartScope`

**Edit:**
- `internal/planparser#Plan`
- `internal/planparser#Plan.Reparse`
- `internal/planparser#checkHandleConsistency`
- `internal/planparser#checkPathMissing`
- `internal/planparser#validate`

**Extends:**
- `internal/planparser/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Intent:** Let the plan checks read one part of a split plan (D7): a part's own plan directory, checked with the exports it may use from other parts counted as declared, and with shared-spot targets refused.

`PartScope` holds `Name` (the part), `Handles []string` (the `plan:` handles other parts and shared spots export to it), `Paths []string` (the worktree-relative new file paths they export to it) and `SpotPaths []string` (every shared spot's worktree-relative path).
`Plan` gains the field `Part *PartScope`, nil for an ordinary plan; a caller sets it after parsing a part directory, and every check below reads it only when it is non-nil, so an ordinary plan's findings do not change.
`Plan.Reparse` carries `Part` onto the plan it returns, so the resolve pass's re-parse after handle canonicalization keeps the scope.

- `checkHandleConsistency`: a referenced handle that `Part.Handles` holds counts as claimed, so it is no `handle-dangling` finding; it never counts toward `handle-collision`, which stays about the plan's own declarations.
- `checkPathMissing`: a ref whose disk path `Part.Paths` holds is satisfied, as a Create target of the plan is.
- `checkPartSpotTarget` implements the new check `part-spot-target`: a ref of any target group of a card (never `Uses`) whose disk path, read as `checkPathMissing` reads it, is one of `Part.SpotPaths`.
  One blocking finding per card and ref, with `Ref` the ref; its detail names the shared spot and the way forward: drop the target and say in the card's `**Intent:**` what the card needs from the spot, since the chair writes the shared spot's cards.
  `validate` runs it right after `checkPathMissing`, and only when `Part` is set.

`TestValidate_PartScope` (tier0) builds part plans through `internal/testkit/plankit` and sets `Part`: a card using a handle `Part.Handles` holds and a path `Part.Paths` holds passes with no `handle-dangling` or `path-missing` finding; the same plan with `Part` nil reports both, which is today's verdict; a handle neither the plan nor `Part.Handles` declares is still `handle-dangling`; a part that declares on one card a handle `Part.Handles` also holds reports no `handle-collision`; a card targeting a spot path under `Edit` is `part-spot-target`, while the same path under `Uses` is not; and a reparsed plan keeps `Part`.

The package doc's check list and `contracts/specs/loom-plan-spec.md`'s validation checks gain the part scope: the field, its effect on `handle-dangling` and `path-missing`, and a `part-spot-target` row, with a note that `lym loom validate-plan --part` is the mode that sets it.

**ImpactSummary:** Adds an optional field read by three checks; with it nil every existing caller's findings are unchanged.
