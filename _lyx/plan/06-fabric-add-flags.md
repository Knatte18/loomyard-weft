# Card 6 — fabric-add-flags

**Edit:**
- `internal/fabriccli#Command`
- `internal/fabriccli#runAdd` -> `func runAdd(ctx context.Context, out io.Writer, args []string, local bool, opts fabricengine.AddOptions) int`
- `internal/fabriccli#addOptionsFromEnv` -> `func addOptionsFromEnv(parent string, adoptLocal bool) fabricengine.AddOptions`

**Create:**
- `plan:internal/fabriccli_test#TestFabricAdd_ParentAndAdoptLocal` -> `func TestFabricAdd_ParentAndAdoptLocal(t *testing.T)`

**Extends:**
- `internal/fabriccli/doc.go`

**Uses:**
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPair`

**Intent:**
`lyx fabric add <slug>` gains `--parent <slug>` (D5) and `--adopt-local` (D6), carried through `addOptionsFromEnv` into `AddOptions.Parent` and `AddOptions.AdoptLocal`; the `add` command's `Long` names both, the parent's dirt rule and the default-branch root pair.
`--local` with either flag is refused before any mutation naming the flag to drop, since a local-only pair takes neither.
Bound: `--adopt-local` is the only route that sets `AdoptLocal`, so no batten, adopt or orch path adopts a stale local branch; `--parent` lets an operator fork from any live pair of the hub, bounded by card 4's refusals.
`TestFabricAdd_ParentAndAdoptLocal` runs the CLI (`RunCLIIn`) from the prime of a hubforge hub: `add member --parent beam` forks from `beam` (origin names it), `add x --adopt-local` adopts a local branch, and `--local --parent beam` is refused naming `--parent`, and `--local --adopt-local` is refused naming `--adopt-local`.

**ImpactSummary:** Two new flags on one command; runAdd's one caller is in Command on the same card.
