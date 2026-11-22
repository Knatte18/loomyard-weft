```yaml
lens: code-tests
outcome: reported
```

My lens is code-tests: for each behaviour the diff adds or changes, I looked for the test row that fails if it breaks. Three behaviours have no such row, and one test's doc says more than it checks.

**Behaviour → pinning test**
- Named `SeedStencil` replaces the shared seed template: `internal/shedadapters/bouncer_seed_test.go:122` `TheBriefFillsTheNamedTemplate`.
- `SeedBrief` fills `seed_brief`; a brief error fails the prompt; a nil brief leaves the marker unfilled: bouncer_seed_test.go:122, :124 and :125.
- An unknown `SeedStencil` fails at construction: bouncer_seed_test.go:157 `AnUnknownSeedStencilFailsConstruction`.
- An empty `SeedStencil` falls back to `bouncer-template-seed` (other recipes unchanged): the existing seed tests (`TestBouncer_SeedCall_HappyPath`, entries_bouncer_test.go:90 and :188), which write `bouncer-template-seed`.
- The shipped `harden-template-seed` marker set and Fill: markers_test.go:90 and bouncer_seed_test.go:123 `TheShippedHardenSeedFills`.
- The recipe names `seed_stencil: harden-template-seed`: hardenerrecipe_test.go:82.
- `bouncerEntry` reads `seed_stencil` and passes it to `BouncerConfig.SeedStencil` (entries_bouncer.go:52, 331): **none**. No shedrecipe test sets the key.
- `bouncerEntry` wires `SeedBrief` from `env.Harden` under `harden_round` (entries_bouncer.go:368): **none**. This is the mutation survivor.
- `Campaign.SeedBrief` with packages, with title and labels, with no labels, and with an entry error: `TestCampaignSeedBrief`, all four rows.
- The production `Entry` seam (hardenwiring.go:168–182): board open error, `GetTask` error, and the "board holds no entry" refusal with its way forward: **none**.
- `CheckCampaign` with no packages passes and runs no copies or tier-test check: campaign_test.go `NoPackagePassesAndRunsNoCopiesOrTierTestCheck`.
- `CheckCampaign` passes the template's tier-0 keys: campaign_test.go `TemplateLiveKeysPass`, with literal tiers.
- Preflight wiring with the template hardener config and no packages: `TestHardenPreflight` `TheTemplateWithNoPackagePasses`. Its fixture has no gate.yaml, so the gate side never runs (see finding 4).
- Preflight wiring refuses a package the worktree lacks: `TestHardenPreflight` `APackageTheWorktreeLacksIsRefused`.
- `hardenLiveKey` without the hub-sets read: `TestHardenLiveKey`.
- Live gate with no packages: no copies, and a pass that started no binary is green: live_test.go `NoPackageRunsNoCopiesAndAPassStartingNoBinaryIsGreen`.
- Live gate with no packages and a failed run is red: live_test.go `NoPackageFailedRunIsRed`.
- Live gate with packages, unchanged behaviour: the existing `TestLiveGate` rows. The packages assertion is now `named`.
- `SeedLocalOrigin` builds one bare holding README on main, and `CloneHub` single-repo clones it: `TestSeedLocalOrigin`.
- The `SeedLocalBares` refactor through `seedReadmeBare`: `TestSeedLocalBares`.
- `sandbox new --single-repo` gives one bare with `refs/lym/`, `single_repo: true`, and is listed and cleared: `TestSandboxVerbs`, the single-repo subtest plus the list and clear subtests.
- `sandbox new` default answers `single_repo: false`: `TestSandboxVerbs` default subtest.
- `sandbox new` default still builds a two-bare hub (the default wiring of `sandboxDeps`): **none**. See finding 5.
- `CloneAndWireFromLocalOrigin`: `TestSandboxVerbs` single-repo subtest, which reaches it in-process through `ExecuteIn`. The reach report missed it because hardencli's map holds no tier1 entry for it.
- Removing `HubSetsGate` and the Preflight hub-sets refusals: the old rows were deleted with the code. Nothing was loosened that still exists.

**Findings**

1. **MEDIUM, design.** `internal/shedrecipe/entries_bouncer.go:368`: no test checks that a `harden_round` Bouncer carries the campaign's `SeedBrief` into the seed prompt. This is the survivor; dropping the line leaves `seed_brief` unfilled, so a real hardener seed fails "bouncer seed prompt fill failed".
   - Evidence: entries_harden_test.go:154–280 covers `harden_round` facts, live-red and halt, but no seed call.
   - Fix: add a row to `TestHardenRoundKey` that sets `seed_stencil` to a stub stencil carrying `{{.seed_brief}}`, runs the seed call, and asserts the campaign's brief text is in the prompt.

2. **LOW, design.** `internal/shedrecipe/entries_bouncer.go:52` and `:331`: no shedrecipe test reads the `seed_stencil` key through to `BouncerConfig.SeedStencil`. Dropping the field assignment at line 331 silently reverts the hardener to the shared seed template.
   - Evidence: grep found no `seed_stencil` or `SeedStencil` in `internal/shedrecipe/*_test.go`.
   - Fix: the same row as finding 1 covers it, because the stub stencil is reachable only through the key.

3. **BLOCKING, design.** `internal/loomcli/hardenwiring.go:179`: the new refusal "the board holds no entry %q; way forward: name the modules … hardener.packages …" in a bound module (`internal/loom*`) has no test that reaches it and asserts its message.
   - This breaks PATTERN-refusal-way-forward. The seam's open-board and `GetTask` error paths are untested too.
   - Evidence: grep for `hardenCampaign(` and `Entry()` in `internal/loomcli/*_test.go` found no test of the seam; the reach report says no mapped test reaches `hardenCampaign`.
   - Fix: add a tier0 test that builds the campaign over a hub fixture with an empty board, calls `Entry()`, and asserts the message holds `hardener.packages` and `lym loom resume`. Add a row for a present entry returning its title and labels.

4. **MEDIUM, consistency.** `internal/loomcli/hardenwiring_test.go:119–121`: `TestHardenPreflight`'s doc says the hub passes "with the template's values". The fixture `hubLocation` (hubfixture_helper_test.go:76–94) writes no gate.yaml, so `gateslot.LoadConfig` returns `ErrConfigAbsent` with a zero `Config` (config.go:227–231).
   - `hardenLiveKey` then returns empty keys, and the row passes without the template's harden gate keys ever being read.
   - A template change putting `harden-live` on the billed tier, or `harden-live-copies` on two tiers, would still pass. The done-when case, a real hub with a sparse gate.yaml and no harden keys, is not pinned.
   - Fix: seed the hub's gate.yaml with a base file holding no harden keys, so the template fill supplies them, and keep the row's assertion.

5. **LOW, design.** `internal/hardencli/cli_integration_test.go:79–112`: the default `new` subtest asserts only `single_repo: false` from the answer, which echoes the flag. It does not check the hub's layout.
   - A wiring slip that applied the single-repo `Seed` and `Clone` by default would still pass, though the focus asks whether the default stays the two-bare hub.
   - Fix: in the default subtest, glob the container's `*-bare` and assert two bares, the mirror of the single-repo subtest's one-bare check.

Tool calls:
- Read `internal/shedrecipe/entries_harden_test.go:150–280` and grepped shedrecipe and loomcli tests for `seed_stencil`/`SeedStencil` and the `Entry` seam. My context lacked the `harden_round` Bouncer tests and whether any test reaches the `Entry` seam.
- Read `internal/hardencli/cli_integration_test.go:64–112`. My context lacked the default-layout subtest's assertions.
