# Card 2 — The launch manifest record and its copy at the run-directory root

**Create:**
- `plan:internal/shuttleengine#Manifest` -> `type Manifest struct`
- `plan:internal/shuttleengine#ManifestItem` -> `type ManifestItem struct`
- `plan:internal/shuttleengine#ManifestChannel` -> `type ManifestChannel string`
- `plan:internal/shuttleengine#ManifestDelivery` -> `type ManifestDelivery string`
- `plan:internal/shuttleengine#ManifestFileName` -> `const ManifestFileName = "manifest.json"`
- `plan:internal/shuttleengine#manifestsDirName` -> `const manifestsDirName = "manifests"`
- `plan:internal/shuttleengine#ErrHookUnrun` -> `var ErrHookUnrun = errors.New("shuttle: the plugin hook is listed unrun")`
- `plan:internal/shuttleengine#WriteManifest` -> `func WriteManifest(runDir string, m Manifest) error`
- `plan:internal/shuttleengine#ReadManifest` -> `func ReadManifest(path string) (Manifest, bool, error)`
- `plan:internal/shuttleengine#Manifest.Omitted` -> `func (m Manifest) Omitted() []ManifestItem`
- `plan:internal/shuttleengine#MissingItems` -> `func MissingItems(from, in Manifest) []ManifestItem`
- `plan:internal/shuttleengine#copyManifest` -> `func copyManifest(root, runID, runDir string) error`
- `plan:internal/shuttleengine#TestManifest_Record` -> `func TestManifest_Record(t *testing.T)`
- `plan:internal/shuttleengine#TestRunner_Start_KeepsTheManifestCopy` -> `func TestRunner_Start_KeepsTheManifestCopy(t *testing.T)`

**Edit:**
- `internal/shuttleengine#Runner.launch`
- `internal/shuttleengine#sweepOrphans`
- `internal/shuttleengine#findRunByStrand`
- `internal/shuttleengine#UserSetup`
- `internal/shuttleengine#InstructionItem`
- `internal/shuttleengine#OmittedHook`
- `internal/shuttleengine#TestSweepOrphans_AgeGuardAndLiveGuid`
- `internal/shuttleengine#TestFindRunByStrand`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#newFixture`
- `internal/shuttleengine#readyAgentEngine`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fakeEngine`

**Intent:**
D1's manifest as an engine-neutral record type, its record file, its comparison and the copy a launch keeps, in `internal/shuttleengine` so both engines and `internal/shuttleparity` share it.

Types:
- `ManifestChannel` constants, one per compared channel of D1 and one for settings:
  `prompt-engine`, `edit-directive`, `standing-passage`, `instruction-source`, `skill-load`, `skill-listing`, `plugin-context`, `guard`, `batching-nudge`, `notice`, `settings`.
- `ManifestDelivery` constants `delivered`, `native` (an item the provider loads itself, listed from the engine's resolution of it) and `omitted`.
- `ManifestItem` holds `Channel`, `Key` (engine-neutral: a heddle slug, a shellguard row name, a skill name, `<plugin>/<hook command>`, an instruction file's path, a fixed notice key), `Engine` (empty for a compared item; the engine's name for an item keyed to one engine, never compared), `Delivery`, `Reason` (why an item was omitted) and `Detail` (free text such as a byte size or a sandbox mode).
  Only an item from a single-engine heddle target and a `settings` item carry an `Engine`; no other item is keyed to one engine, so there is no exemption table.
- `Manifest` holds `Engine`, `Role`, `PromptEngine` and `Items`.
- `ErrHookUnrun` is the error a hook runner returns for a hook it lists without running; card 4 keeps it out of Codex's per-hook warning and card 6's `UnrunHooks` returns it.

Functions:
- `WriteManifest` writes `<runDir>/manifest.json` through `state.WriteJSON`, and `ReadManifest` reads a record through `state.ReadJSON`, `found` false for an absent file; a record with a key it does not know still reads (`PATTERN-tolerant-records`).
- `Manifest.Omitted` is the record's verdict: every item whose `Delivery` is `omitted`, so a plugin context left out at run time fails it.
- `MissingItems(from, in)` returns each compared item of `from` (empty `Engine`, channel not `settings`) whose channel and key `in` holds no item of, whatever that item's delivery; the comparison is by key, so each engine words its copy its own way.

The copy: `Runner.launch`, right after `saveRunState` succeeds, calls `copyManifest(root, runID, runDir)`.
It reads `<runDir>/manifest.json` and, when found, writes it as `<root>/manifests/<runID>.json`; an absent record writes nothing, so an engine that writes none (a test fake) launches as before.
A copy placed after the run record is saved belongs to a launch whose strand registered, so an attempt that fails before then, or a start retry's failed attempt, leaves no copy.
A record that cannot be read or a copy that fails to write is logged at Warn and never fails the launch, since the run-directory record stands.
A done run's cleanup removes only the run directory, so the copy stays; bound: nothing removes the copies.
`lym shuttle manifest` calls `Prepare` directly and never `Runner.launch`, so it keeps no copy (D10).

`sweepOrphans` and `findRunByStrand` skip the entry named `manifestsDirName`, since it holds no `run.json`: the sweep no longer removes it once it is old, and the lookup no longer counts it as an unreadable run directory.
The other scans of the run-directory root already skip a directory with no `run.json` silently.

The seam types gain what the manifest keys on: `UserSetup` gains `ListedSkills []string`, the names its `SkillListing` lists, in listing order;
`InstructionItem` and `OmittedHook` gain `Plugin` and `Hook`, set on a plugin session-start context to its plugin id and hook command, empty on every other item.
Card 3 fills them.

Tests, all tier0:
- `TestManifest_Record` is a table: a round trip through `WriteManifest` and `ReadManifest`; a record file holding an unknown key reads with its known fields; an absent file reads not found;
  `Omitted` returns exactly the omitted items; `MissingItems` returns a compared item absent from `in`, passes an item present under another delivery (an omitted one included), and skips an item keyed to one engine and a `settings` item.
- `TestSweepOrphans_AgeGuardAndLiveGuid` seeds a `manifests` directory aged past the guard and asserts it survives, its `removed` expectation unchanged.
- `TestFindRunByStrand` gains a row: a root holding only a `manifests` directory answers the not-found error wrapping `ErrNoRunRecord`, not the unreadable-directories one.
- `TestRunner_Start_KeepsTheManifestCopy` is a table driving a run with `newFixture` and a `fakeEngine` whose existing `PrepareHook` writes a manifest with `WriteManifest`, to done with `KeepPane` false:
  the run directory is gone and `<root>/manifests/<runID>.json` reads back the hook's manifest;
  a hook that writes none leaves no `manifests` directory;
  with a regular file already at `<root>/manifests`, the copy fails, the run still starts and finishes done, and no copy exists.

`doc.go` gains a paragraph on the manifest record, its copy and the two scans that skip the copies' directory.

**ImpactSummary:** New types and two record functions in `shuttleengine`; `Runner.launch` gains one non-fatal copy step after its run record; two root scans skip one directory name; three seam structs gain fields no existing caller sets.

**Commit:** 2: Write the launch manifest record and keep its copy at the run-directory root
