# Card 1 — Standing passages and heddle file entries carry their slug

**Edit:**
- `internal/heddleengine#Renderer.Passages` -> `func (r Renderer) Passages(boltDir, engine, role string) ([]Passage, error)`
- `internal/heddleengine#passagesOf` -> `func passagesOf(layers Layers, engine string, targets ...string) []Passage`
- `internal/heddleengine#Renderer.Show`
- `internal/heddleengine#targetTable`
- `internal/heddleengine#TestShowAndPassagesReadTheShippedLayerFirst`
- `internal/heddleengine#TestRender`
- `internal/heddleengine/doc.go`
- `internal/shuttleengine#PassageReader` -> `type PassageReader func(boltDir, engine, role string) (Standing, error)`
- `internal/shuttleengine#StandingPassages` -> `func StandingPassages(boltDir, engine, role string) Standing`
- `internal/shuttleengine#InstallPassageReader`
- `internal/shuttleengine#installedPassageReader`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#TestPrepare_AppendsStandingPassages`
- `internal/shuttleengine/codexengine#Codex.Prepare`
- `internal/shuttleengine/codexengine#passageItems` -> `func passageItems(passages []shuttleengine.Passage) []instructionItem`
- `internal/shuttleengine/codexengine#TestPrepare_AppendsStandingPassages`
- `cmd/lym#newRoot`

**Create:**
- `plan:internal/heddleengine#Passage` -> `type Passage struct`
- `plan:internal/heddleengine#Renderer.FileEntries` -> `func (r Renderer) FileEntries(boltDir, engine string) ([]Passage, error)`
- `plan:internal/heddleengine#TestRenderer_EntryKeys` -> `func TestRenderer_EntryKeys(t *testing.T)`
- `plan:internal/shuttleengine#Passage` -> `type Passage struct`
- `plan:internal/shuttleengine#Standing` -> `type Standing struct`
- `plan:internal/shuttleengine#TestStandingPassages` -> `func TestStandingPassages(t *testing.T)`

**Uses:**
- `internal/heddleengine#promptTargetsFor`
- `internal/heddleengine#reachesFileTarget`

**Intent:**
D1 keys standing passages by heddle slug and heddle file entries by the shared or single-engine target they reach through, so the reader seam must carry the slug and the target, not the text alone.

In `internal/heddleengine`, a new `Passage` holds `Slug`, `Text` and `Engine`.
`Engine` is empty when the entry reaches the session through a shared target (`agents-prompt`, `agents-file`) or an engine-neutral one (`orch-prompt`), so it is compared;
it names the engine when the entry reaches it only through a target aimed at that engine (`claude-prompt`, `codex-prompt`, `claude-file`, `codex-file`), so it is not compared.
An entry aimed at both a shared target and a single-engine target reaches the session through the shared one, so its `Engine` is empty.
`Renderer.Passages` returns `[]Passage` in the same render order and with the same one-entry-once rule; `passagesOf` takes the session's engine to fill `Engine` from the entry's own `Targets`.
`Renderer.Show` joins the texts of the `Passage` values `passagesOf` returns for the named prompt target, passing an empty engine, so its output keeps its bytes.
The `claudeengine` and `codexengine` `doc.go` files stay as they are for this card, and cards 3 and 4 cover them.
The `claude-file` and `codex-file` rows of `targetTable` gain `Engine: "claude"` and `Engine: "codex"`; `promptTargetsFor` skips file rows before it reads `Engine`, so no prompt reach changes.
`Renderer.FileEntries(boltDir, engine)` returns the entries that render into the file target whose `Engine` is engine, through `reachesFileTarget`, keyed the same way; an engine with no file target gets none.
It reads the layers through the same locked read `Passages` uses and writes nothing.

In `internal/shuttleengine`, `Passage` mirrors the heddle one (`Slug`, `Text`, `Engine`), and `Standing` holds `Passages []Passage` and `FileEntries []Passage`.
`PassageReader` returns a `Standing`; `StandingPassages` returns it, and the zero `Standing` for an empty Bolt dir, no installed reader or a reader error (still logged at Warn).
`InstallPassageReader` keeps its name, so `cmd/lym/prerunseams_test.go`'s check of the install still holds.

`cmd/lym#newRoot` installs a closure that calls `heddleengine.Renderer{}.Passages` and `FileEntries` and converts both into a `shuttleengine.Standing`; heddle does not import shuttleengine.
`Claude.Prepare` and `Codex.Prepare` take `.Passages` and pass each passage's text where they passed the string before, so the appended prompt and the developer instructions keep their bytes;
`passageItems` names each item `heddle passage <slug>` in place of its position.

Tests, all tier0:
- `TestRenderer_EntryKeys` is a table over a fixture store with one entry per target kind and one entry aimed at both `agents-prompt` and `claude-prompt`.
  It asserts each returned `Passage`'s `Slug` and `Engine` from both `Passages` and `FileEntries` for each engine, the double-aimed entry once with an empty `Engine`, and that `FileEntries` returns none for an unknown engine.
- `TestShowAndPassagesReadTheShippedLayerFirst` and `TestRender` read the texts off the returned `Passage` values; their expectations keep the same texts.
- `TestStandingPassages` is a table: an empty Bolt dir, no installed reader and a reader that errors each give the zero `Standing`; an installed reader's `Standing` comes back whole.
  It installs the process reader, so it does not run in parallel.
- Both `TestPrepare_AppendsStandingPassages` tests install readers returning a `Standing` and keep their assertions on the launch text.
  The Codex test's oversize-refusal row changes its expected item name from `heddle passage 1` to `heddle passage <slug>` with its reader's slug.

**ImpactSummary:** One seam type re-signed with three production callers (two engine Prepares and `cmd/lym`'s install) and five tests; launch text is unchanged and only the Codex refusal's item name changes.

**Commit:** 1: Standing passages and heddle file entries carry their slug
