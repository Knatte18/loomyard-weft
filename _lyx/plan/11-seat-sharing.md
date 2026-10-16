# Card 11 — seat-sharing

**Edit:**
- `internal/seatengine#seatSpec`
- `internal/seatengine#TestSeatPrompts_ValuesPromptsAndSpecs`
- `internal/seatengine/doc.go`

**Intent:**
Build D6: every strand a seat table launches, the chair and every advisor, declares the sharing kind, so seat agents that run side by side split the free rows evenly and the driver collapses while they are live (card 1).
`seatSpec` sets `Display: render.Display{Size: render.Size{Kind: render.SizeSharing}}`; its anchor stays empty, which shuttle's `Validate` turns into below-parent, and shuttle passes `Display` to reed unchanged (`run.go` hands `spec.Display` to `reedengine.AddSpec`), so reed stays a dumb carrier.
`seatengine/doc.go` states that seat strands share rows.

Test: `TestSeatPrompts_ValuesPromptsAndSpecs` asserts that the chair spec and the advisor spec both carry the sharing kind and no anchor.
The heights two and three sharing strands get, and the bottom-most positional strand expanding again once none is live, are card 1's `TestRulesGolden` rows.

**ImpactSummary:** one spec field on the seat spec builder; every seat table caller gets sharing strands, and no other spec field changes.
