# Card 19 — agentname task slug

**Create:**
- `plan:internal/agentname#TaskSlug` -> `func TaskSlug(strandName string) string`
- `plan:internal/agentname#TestTaskSlug` -> `func TestTaskSlug(t *testing.T)`

**Edit:**
- `internal/configcli#hubWriteRefusal`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Lift the slug half of the hub-wide config write guard into the one function D8 requires the board's `run_settings` guard to share with it.
`TaskSlug` returns the slug of a strand name that parses through `Parse` and carries one, and the empty string for an empty name, a slug-free name such as the hub orch's, and a name that does not parse; `agentname` stays standard-library only (`PATTERN-agent-name`).
`hubWriteRefusal` calls it in place of its own `Parse` and slug test, its message and its prime check unchanged; the guard keeps its stated bound, refusing a mistaken write and not a determined one.

`TestTaskSlug` is table-driven: a full `<shortname>:<slug>:<role>` name yields its slug, the orch's slug-free name, an empty name and an unparseable one yield empty.
The budget row for `internal/agentname` rises by one.

**ImpactSummary:** one new agentname function; configcli's hub write guard calls it with unchanged behaviour.
