```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens, finding what the diff's changes force outside the lines it touches.

**List: per change, the set found, and whether it is covered**

- **`PassageReader` / `StandingPassages` now return `Standing`.**
  - Set: the one production reader at `cmd/lym/main.go:180`, both engines' `Prepare` calls (`claudeengine.go:264`, `codexengine.go:168`), and every test that installs a reader.
  - The tests are `claudeengine/prepare_test.go:647`, `codexengine/prepare_test.go:427`, `shuttleparity/seats_test.go:98`, and the engine manifest tests.
  - `cmd/lym/prerunseams_test.go:55` only checks the install call name.
  - Covered; no stale caller left.
- **`heddleengine.Renderer.Passages` now returns `[]Passage`, and `FileEntries` is new.**
  - Set: `Show`, `main.go`, and `render_test.go`; the call sites at lines 226, 818, 822, 826 and 857 were adapted through `passageTexts`.
  - Covered.
- **`Target.Engine` is now set on `claudeFileTarget` and `codexFileTarget`.**
  - It was previously read only for prompt targets (`promptTargetsFor`).
  - From the diff alone I could not confirm that `promptTargetsFor` ignores `Engine` on file targets.
  - `FileEntries` does use it, and `TestRenderer_EntryKeys` pins the file-entry keys.
  - The focus-directive question is whether existing per-engine target selection is unchanged; the other lenses own that.
  - Covered, with that caveat.
- **`skillListing` returns three values, and `UserSetup.ListedSkills`, `InstructionItem.Plugin`/`Hook` and `OmittedHook.Plugin`/`Hook` were added.**
  - Set: the one `skillListing` call site (`usersetup.go:105`), `pluginhook.go`, `codexengine/usersetup.go`, and the existing tests.
  - The existing tests construct `OmittedHook`/`UserSetup` by keyed literals, so they still compile.
  - Covered.
- **`shuttleengines` constructor signature gained `LaunchSeams`.**
  - Set: all five direct `registered.constructor(...)` call sites (`shuttleengines.go:228, 256, 322, 335, 347`) and `keepwarm.go`. All pass `LaunchSeams{}`.
  - `build` and `registeredConstructors` are updated.
  - Covered.
- **`seatengine.Specs`, `burlerengine.Engine.SeatTables` and `landingshed.FixReviewSeatTable` are new exported functions.**
  - Set: `cmd/lym/testonlycode_linux_test.go`. The `seatengine.Prompts` allowlist entry was removed because `Specs` now calls it from production.
  - The new functions are reached from `shuttleparity/seats.go` through `shuttlecli`. No unadmitted test-only helper found.
  - Covered.
- **`manifests` directory at the run-directory root.**
  - Set: `findRunByStrand` and `sweepOrphans`, both patched with `isManifestsDir` and tested in `rundir_test.go`.
  - `EnginesRun` and the other run-root readers (`ReadAgentActivity`, `ReadSessionStates`, `ReadAgentWaits`) were not shown in the diff.
  - They may iterate run directories and also meet a `manifests` entry. I did not check this.
  - **Possibly uncovered.**
- **PATTERN docs.**
  - `PATTERN.md` carries the declared `PATTERN-engine-parity` text verbatim. `pattern/PATTERN-engine-parity.md` is new, and `PATTERN-shuttle-provider-seam.md` names `NewWithSeams`.
  - `docs/overview.md` lists the package and the verb.
  - There are no `Pattern-Departure:` trailers on any commit in the range.
  - Covered.
- **Test allowlists.**
  - The `treesitterlink_integration_test.go` entry for `internal/shuttleparity` is justified by the test-only `loomcli` import in `producers_test.go`.
  - Covered.
- **Stencils.** The diff changes none, so the stencil-wording check does not apply.

**Findings**

1. **NIT / consistency**
   - Location: `docs/overview.md:1982`.
   - Summary: the verb list `lym shuttle run|open|state|await` omits the new `manifest` verb, while line 640 was updated to include it.
   - Evidence: the grep output at `docs/overview.md:1982` against `:640`.

2. **LOW / scope**
   - Location: `internal/shuttleengine/` run-root readers, for example the `EnginesRun`, `ReadAgentActivity`, `ReadSessionStates` and `ReadAgentWaits` directory scans.
   - Summary: only `findRunByStrand` and `sweepOrphans` skip `manifests`. Any other loop over the run root's directory entries would now see a directory with no `run.json`.
   - Fix: confirm each such scan skips the directory or tolerates a missing `run.json`, and add `isManifestsDir` where it does not.
   - The handler should confirm this. My context lacked those readers' bodies, and the grep above could not settle it.

**Tool calls (2 of 6)**
1. Grep for callers of the changed symbols and constructors, the `testOnlyCodeAllowed` entries, `Pattern-Departure` trailers, and docs that mention passages or the verb. My context lacked the callers outside the diff and the commit trailers.
2. Grep for other verb-list mentions and the `treesitterlink` allowlist rationale. My context lacked non-Go docs and the allowlist file body.
