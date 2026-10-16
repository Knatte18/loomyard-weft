---
round: 1
focus:
  - "Card 2, test economy: it adds an `unknown flag exits non-zero` subtest and extra stderr and heading assertions beside table rows. Settle whether an existing test in `tools/sandbox` already covers each of these behaviours, and name that test if so."
  - "Card 2, `Edit` member glyphs: its Intent also rewrites the header comments of `main.go` and `suite.go`, and the `suiteSpec` doc comment. Settle whether the glyph list names the members whose bodies or comments change, using `lyx quarry resolve` to confirm spans."
  - "Card 2, `Create` glyphs: check that `creelSandboxSuiteMD` and `creelSuite` as separate `Create` targets fit the granularity rule."
  - "Card 1, fidelity: check each D1 and D3 item (preamble sections, `ref` bullet, session log for S1 to S8 and for S9 plus C1 to C13, Notes pointer, S8 as last scenario) against what the card says. Check that no card adds scope outside the decision record."
  - "Card 1, board note: confirm it writes only the note entry `creel-sandbox-c13-stacked-rewrite` and never the task's own entry `creel-sandbox-suite`. Also settle whether the `lyx board upsert` invocation shape matches `lyx board --help`."
  - "Card order: card 3's docs link and card 2's `go:embed` both depend on a file card 1 creates. Confirm the order holds, and that card 3's `Uses` list is sound."
  - "`## verify:` coverage: settle that `tools/sandbox` and `cmd/lyx` (the coverage guard) tests are run by the listed commands, and that the section follows the stencil's order."
  - "Attack-surface and refusal reachability in card 2: the new subcommand's stated bound, and whether the named inputs (absent hub, `-bogus` flag) reach the prescribed refusals without an upstream check rejecting them first."
---

Seed focus from the plan overview, the three cards and the decision record.
No item restates a gate check or an entry on the `Do not flag` list.
