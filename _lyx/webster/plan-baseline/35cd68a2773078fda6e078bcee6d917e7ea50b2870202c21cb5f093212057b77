# Card 12 — board-parent-beam

**Create:**
- `plan:internal/boardengine#ParentBeam` -> `func ParentBeam(tasks []Task, slug string) string`
- `plan:internal/boardengine#ReadTasks` -> `func ReadTasks(boardDir string) ([]Task, error)`
- `plan:internal/boardengine#Board.ParentBeamOf` -> `func (b *Board) ParentBeamOf(slug string) (string, error)`
- `plan:internal/boardengine_test#TestParentBeam` -> `func TestParentBeam(t *testing.T)`

**Edit:**
- `internal/boardengine#BriefTask`
- `internal/boardengine#Store.ListTasksBrief`
- `internal/boardengine_test#TestListTasksBriefLayerAndProposal`

**Extends:**
- `internal/boardengine/doc.go`

**Uses:**
- `plan:internal/boardengine_test#newProjectBoard`

**Intent:**
D4's board half.
`ParentBeam` walks the entry's `project` links upward, skipping done projects and grouping projects (`beam` false), and returns the slug of the nearest open project whose `beam` is true, or the empty slug for the prime: no project, no beam, only done beams, or a slug the list does not hold.
A cycle cannot be stored (card 10), and the walk still stops after visiting every entry once, returning the empty slug.
`ReadTasks` reads the local `board.json` under its swap lock with no write and no legacy fold beyond `Load`'s, for readers outside a `Board` such as hubgeom's `ParentOf` (card 15); an absent file is an empty list.
`BriefTask` gains `ParentBeam` (`parent_beam`): `ListTasksBrief` fills it from the store's `parentOf` seam when set and from `ParentBeam` over the store's own entries otherwise, empty for the prime; a resolver error leaves it empty and the list still succeeds, as the owner cell does.
`Board.ParentBeamOf` answers the same for one slug, for `lyx board get` (card 17).
`TestParentBeam` rows: no project is the prime; a task under a grouping project under a beam returns the beam; a member whose only beam is done returns the prime; a grouping project alone returns the prime; nested beams return the nearest; a `project` chain with a cycle, written into the list by hand, returns the prime;
and, over a `Board` from `newProjectBoard`, `ListTasksBrief` and `ParentBeamOf` carry the `parent_beam` a `Config.ParentOf` fake answers, and a fake that errs leaves `parent_beam` empty while the list still succeeds.
`TestListTasksBriefLayerAndProposal` gains a row, over its bare store with no seam, whose `parent_beam` reads `ParentBeam`.
The package doc's Owners section names `ParentBeam` as the board half of the one parent resolver.

**ImpactSummary:** Adds one list field and three read-only functions; no existing caller's behaviour changes.
