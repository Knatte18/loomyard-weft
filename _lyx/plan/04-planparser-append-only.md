# Card 4 — planparser-append-only

**Create:**
- `internal/planparser/appendonly.go`
- `internal/planparser/appendonly_test.go`

**Uses:**
- `internal/planparser/plan.go`

**Intent:**
Add the comparison helper the rework row's append-only check calls (D5), in the parser package so the comparison runs over `planparser`'s own model.
`CheckAppendOnly(base, extended *Plan) []string` returns one human-readable violation per breach, and nil when `extended` is `base` plus one or more appended cards:
- a changed frontmatter field (`Format`, `Approved`, `Root`, `Language`), each named;
- a changed `Framing`, `SharedDecisions`, `RenameMechanic` or `Verify`, each named by its section;
- a pre-existing card (by position in `base.Cards`) missing from `extended` or differing from its base form, named `card N (<slug>)`;
  compare cards field by field with `reflect.DeepEqual` over the parsed `Card`, so a whitespace-only reflow the parser normalizes away is not a violation;
- zero appended cards (`len(extended.Cards) <= len(base.Cards)`).
Card numbering after the base cards is already enforced by `ValidateFormat`, so the helper does not re-check it.
`appendonly_test.go` is table-driven over parsed fixture plans written to `t.TempDir()`: one appended card and two appended cards each return nil;
an edited existing card's intent, a flipped `approved:`, an edited `## verify:` section, an edited framing paragraph, and no new card each return exactly the violation naming it.
