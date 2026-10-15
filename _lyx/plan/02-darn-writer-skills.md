# Card 2 — darn-writer-skills

**Edit:**
- `internal/darnengine#darnSkills`
- `internal/darnengine#DarnInputs`
- `internal/darnengine#DarnTable`
- `internal/darnengine#TestDarnSpec`

**Uses:**
- `plan:internal/languageconfig#AppendSkills`

**Intent:**
The darn writer gains `scribe:testing` (D6) and loads the told language list after its own (D4).

`darnSkills` becomes `scribe:prose`, `scribe:code-quality`, `scribe:testing`.
`DarnInputs` gains the field `LanguageSkills []string`, documented as the hub's language skills the writer loads after its own, empty for none; it is optional, so `DarnTable`'s required-field check does not name it.
`DarnTable` sets the lead seat's `Skills` to `languageconfig.AppendSkills(darnSkills, in.LanguageSkills)`.

`TestDarnSpec` moves its skill assertion from the shared loop into a per-row `wantSkills`:
the two existing rows, which tell no language list, want `scribe:prose`, `scribe:code-quality`, `scribe:testing`;
a new row tells `LanguageSkills` `scribe:golang-testing`, `scribe:testing`, `scribe:golang-testing` and wants `scribe:prose`, `scribe:code-quality`, `scribe:testing`, `scribe:golang-testing`, so the list follows the writer's own and a repeat or an own skill appears once.

**ImpactSummary:** one optional input field and one list literal in `darnengine`; the only production caller, `internal/loomcli`'s `wireDarn`, compiles unchanged and tells the list in card 5.
