# Card 2 — The launch manifest record and its copy at the run-directory root

**Create:**
- `internal/shuttleengine#Manifest`
- `internal/shuttleengine#ManifestItem`
- `internal/shuttleengine#ManifestChannel`
- `internal/shuttleengine#ManifestDelivery`
- `internal/shuttleengine#ManifestFileName`
- `internal/shuttleengine#manifestsDirName`
- `internal/shuttleengine#ErrHookUnrun`
- `internal/shuttleengine#WebsterForkDenyKey`
- `internal/shuttleengine#WriteManifest`
- `internal/shuttleengine#ReadManifest`
- `internal/shuttleengine#Manifest.Omitted`
- `internal/shuttleengine#MissingItems`
- `internal/shuttleengine#copyManifest`
- `internal/shuttleengine#TestManifest_Record`
- `internal/shuttleengine#TestRunner_Start_KeepsTheManifestCopy`

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
- `cmd/lym#testOnlyCodeAllowed`

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
- `Manifest` holds `Engine`, `Role`, `PromptEngine`, `Items` and `Verdict []ManifestItem`, the record's verdict.
- `WebsterForkDenyKey` is the `guard` key of the `lym webster` fork deny; card 3 lists it, and card 6's comparison gates it on the fork requirement.
- `ErrHookUnrun` is the error a hook runner returns for a hook it lists without running; card 4 keeps it out of Codex's per-hook warning and card 6's `UnrunHooks` returns it.

Functions:
- `WriteManifest` sets the record's `Verdict` from `m.Omitted()` and writes `<runDir>/manifest.json` through `state.WriteJSON`, and `ReadManifest` reads a record through `state.ReadJSON`, `found` false for an absent file; a record with a key it does not know still reads (`PATTERN-tolerant-records`).
- `Manifest.Omitted` returns every item whose `Delivery` is `omitted`.
  Since every `Prepare` writes through `WriteManifest`, each record in `manifest.json` stores that list as its `Verdict`, so a plugin context left out at run time fails the verdict the bench report reads.
- `MissingItems(from, in)` returns each compared item of `from` (empty `Engine`, channel not `settings`) whose channel and key `in` holds no item of, whatever that item's delivery; the comparison is by key, so each engine words its copy its own way.

The copy: `Runner.launch`, right after `saveRunState` succeeds, calls `copyManifest(root, runID, runDir)`.
It reads `<runDir>/manifest.json` through `ReadManifest` and, when found, writes it as `<root>/manifests/<runID>.json`; an absent record writes nothing, so an engine that writes none (a test fake) launches as before.
A copy placed after the run record is saved belongs to a launch whose strand registered, so an attempt that fails before then leaves none.
An attempt that fails after its run record is saved, such as one whose startup the start retry relaunches, keeps its copy under its own run id.
A record that cannot be read or a copy that fails to write is logged at Warn and never fails the launch, since the run-directory record stands.
Each warning names the path and the error, with its own way forward.
For an unreadable record it names the run directory's `manifest.json`, with the way forward: when the file does not decode, fix the engine's `shuttleengine.WriteManifest` call in its `Prepare`, and when it cannot be read, fix the run directory's `manifest.json` permissions.
For a failed copy write it names the copy's path, with the way forward: remove whatever blocks `<root>/manifests`, or make it a writable directory with free space, so the next launch keeps its copy.
A done run's cleanup removes only the run directory, so the copy stays; bound: nothing removes the copies.
`lym shuttle manifest` calls `Prepare` directly and never `Runner.launch`, so it keeps no copy (D10).

`sweepOrphans` and `findRunByStrand` skip the entry named `manifestsDirName`, since it holds no `run.json`: the sweep no longer removes it once it is old, and the lookup no longer counts it as an unreadable run directory.
The other scans of the run-directory root already skip a directory with no `run.json` silently.

The seam types gain what the manifest keys on: `UserSetup` gains `ListedSkills []string`, the names its `SkillListing` lists, in listing order;
`InstructionItem` and `OmittedHook` gain `Plugin` and `Hook`, set on a plugin session-start context to its plugin id and hook command, empty on every other item.
Card 3 fills them.

Tests, all tier0:
- `TestManifest_Record` is a table: a round trip through `WriteManifest` and `ReadManifest`; a record file holding an unknown key reads with its known fields; an absent file reads not found;
  `Omitted` returns exactly the omitted items; a record written with omitted items reads back with a `Verdict` holding exactly those items, and one with none reads back with an empty `Verdict`; `MissingItems` returns a compared item absent from `in`, passes an item present under another delivery (an omitted one included), and skips an item keyed to one engine and a `settings` item.
- `TestSweepOrphans_AgeGuardAndLiveGuid` seeds a `manifests` directory aged past the guard and asserts it survives, its `removed` expectation unchanged.
- `TestFindRunByStrand` gains a row: a root holding only a `manifests` directory answers the not-found error wrapping `ErrNoRunRecord`, not the unreadable-directories one.
- `TestRunner_Start_KeepsTheManifestCopy` is a table driving a run with `newFixture` and a `fakeEngine` whose existing `PrepareHook` writes a manifest with `WriteManifest`, to done with `KeepPane` false:
  the run directory is gone and `<root>/manifests/<runID>.json` reads back the hook's manifest;
  a hook that writes none leaves no `manifests` directory;
  with a regular file already at `<root>/manifests`, the copy fails, the run still starts and finishes done, and no copy exists;
  a hook that writes a `manifest.json` that is not JSON leaves the record unreadable, and the run still finishes done with no copy.

`doc.go` gains a paragraph on the manifest record, its copy and the two scans that skip the copies' directory.

`testOnlyCodeAllowed` in `cmd/lym/testonlycode_linux_test.go` gains the entry `internal/shuttleengine.MissingItems`, its `Why` saying it has no caller until `lym shuttle manifest` lands and that card removes the entry, as the `internal/runfsm.Order` entry does for its own later card.
Its one production caller is card 6's `shuttleparity.Compare`, which no main package reaches before card 8, so without the entry a batch gate ending before card 8 fails `PATTERN-test-only-code`'s guard.

**ImpactSummary:** New types and two record functions in `shuttleengine`; `Runner.launch` gains one non-fatal copy step after its run record; two root scans skip one directory name; three seam structs gain fields no existing caller sets; `testOnlyCodeAllowed` gains an interim entry for `MissingItems`, which card 8 removes.

**Commit:** 2: Write the launch manifest record and keep its copy at the run-directory root
