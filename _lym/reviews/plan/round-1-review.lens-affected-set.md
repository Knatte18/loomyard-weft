```yaml
lens: affected-set
outcome: reported
```

Lens affected-set: for each change a card makes, I listed what reads or asserts it and which cards cover that.

**List: per change, the set found and the cards that cover it**

1. **Card 2 adds the gate template keys `harden-live`, `harden-live-copies` and `harden-burler-fix`.**
   - Set: `internal/gateslot/template.yaml`; `goReadGates` (`tiers.go:161`); `templateDefaults` and `TestConfig_TierUse` (`config_test.go`); `internal/gateslot/doc.go:109`, `:117-119` and the site table at `:258-270`; the key and site table in `pattern/PATTERN-test-speed.md:48-58`; `pattern/PATTERN-gate-slot.md:105`; the example hub files in `docs/dotnet-hub.md:121` and `internal/gatecli/doc.go:987`; the fake gate tiers at `internal/testkit/envkit/envkit.go:322`.
   - Covered: card 2 covers the template, `goReadGates`, `templateDefaults` and `TestConfig_TierUse`, and only *Extends* `gateslot/doc.go`.
   - Not covered: the claims at `doc.go:109` and `:119` and the site table, which need rewriting rather than additions; and `PATTERN-test-speed.md`'s key table.
   - The example files are not closed lists, so they need no change.
2. **Card 4 adds `HardenFix` to `LoomBuildEngines` and the engine choice text.**
   - Set: `internal/shedbuild/loomcontract.go:34` (the literal) and `build_test.go:198,203`.
   - Covered by card 4 (`checkLoomContract` and `TestBuild_LoomContract`).
3. **Card 13 admits `hardener` to the closed recipe lists.**
   - Set: `shedrun/seed.go:94`; `battencli/wire.go:607,613`; `loomrecipe/names.go:30,140`; `finalizerow.go:17`; `bouncerrows.go:18`; `reportengine/runs.go:26`; `timingspans.go:76`; `shedcli/table.go`.
   - All covered by card 13.
   - The accessors in `loomrecipe/recipekeys.go` read the embedded recipe itself, so they hold no closed list and need no change.
   - Uncovered: the board CLI help text at `internal/boardcli/cli.go:604`, which lists the `recipe` values.
4. **Cards 18 and 19 delete treadle.**
   - Set: `shuttleparity/producers_test.go` and `cmd/lym/pathrules_test.go` (card 18); `contracts/stencils/stencils.go`, `agentconfig/config.go` and `template.yaml`, `.gitattributes`, `verifyrun/run.go:31`, `webstercli/sync.go:5`, `shedengine/seam_enforcement_test.go:4` and `pattern/PATTERN-cwd-resolution.md:40` (card 19); `docs/overview.md:326,2105-2106` and `docs/reference/weaving-glossary.md:51,65,95,117,36` (card 20); `internal/mergeresolve/seam_enforcement_test.go:26`.
   - Not covered: the `mergeresolve` comment.
5. **Card 20 deletes crucible.**
   - Set: `fabricvocab.go:32`, `PATTERN-fabric-boundary.md:20`, `tools/codestats/main.go:8`, `scan.go:270` and the `docs/overview.md` links: covered.
   - The history comments in `webstercli/cli_tmux_test.go:72` and `loomshed/seed_test.go:116`, the `crucible:` subject string in `fabriccli/mergescenario_integration_test.go:310`, and the `docs/research` and `docs/benchmarks` files fall under the kinds card 20 keeps.
6. **Card 3 registers the `hardener` module.**
   - Set: `configreg.Modules` and its three tests: covered by card 3.
7. **Card 16 adds the `harden` CLI module.**
   - Set: `newRoot`, the root's `Long` module list, `cmd/lym/registration_test.go` and `clitree_test.go`: covered by card 16; the walk tests pick the module up on their own.
8. **Card 1 adds the started-binaries line to `lym gate test` output.**
   - I found no other parser of the line, and could not establish that any exact-output assertion reads it.

**Findings**

- **MEDIUM, consistency.** Location: card 2 (`02-gate-harden-keys.md`), `internal/gateslot/doc.go` under **Extends:**.
  - Summary: card 2 covers only part of what its new keys force.
    - `doc.go:109` says the template gives `webster-fix`, `darn-write`, `code-burler-fix` and `pr-review` the `diff` scope. `doc.go:119` says `mutation` is listed last on "every key of scope `diff`" and names those four keys. Three new `diff` keys falsify both, and an Extends cannot rewrite them; the doc's recipe-to-site-to-key table at `:258-270` also gains no rows.
    - The card does not say whether the new `diff` keys carry the template's `reach`, `redundancy` or `mutation` reports.
    - `pattern/PATTERN-test-speed.md:48-58`, the background table of every gate key and its site, is listed by no card.
  - Fix: move `internal/gateslot/doc.go` under **Edit:** on card 2, state the new keys' `reports` in the Intent, and add `pattern/PATTERN-test-speed.md` to a card.
  - Evidence: `internal/gateslot/doc.go:109,117-119,258-270`; `pattern/PATTERN-test-speed.md:48-58`.
- **LOW, consistency.** Location: card 19 (`19-treadle-delete.md`), its list of comments that name treadle.
  - Summary: card 19 rewords the treadle mentions in `waitDelay`, `sync.go` and the header of `shedengine`'s seam test. It misses the doc comment on `mergeresolveAllowedImports` in `internal/mergeresolve/seam_enforcement_test.go:26` (package `mergeresolve`). That comment says "the same call treadleengine's seam allowlist already makes" and goes stale once card 19 deletes the package.
  - Fix: add `internal/mergeresolve#mergeresolveAllowedImports` to card 19's **Edit:** list.
  - Evidence: `internal/mergeresolve/seam_enforcement_test.go:26`.
- **LOW, consistency.** Location: card 13 (`13-hardener-recipe.md`).
  - Summary: D1 puts "its board recipe value" in scope, but no card updates the board CLI help that lists the `recipe` values.
  - The list is already incomplete (it omits sampler, pawl and raddle), and the board validates nothing against it.
  - Evidence: `internal/boardcli/cli.go:604`.

**Tool calls**

1. Grepped the readers of `code-burler-fix`, the recipe-name sites and the engine choice text, because the manifest lacked the full set of each.
2. Grepped the board's recipe vocabulary, gate-key lists in docs and the `RecipeDarn` sites, because the manifest lacked the board and doc readers.
3. Read `loomrecipe/recipekeys.go`, the `gateslot` retired-list table, the `PATTERN-test-speed` and `PATTERN-gate-slot` key tables and the `gateslot/doc.go` claims, because they were not in the manifest.
4. Grepped the `recipekeys.go` accessors and readers of gate output, to check whether the recipe-key lookup holds a closed list and whether anything else parses gate output.
5. Listed the `websterengine` verify parser and the `gatecli` exact-output tests, to check whether the started line breaks an existing assertion; the result was inconclusive and grounds no finding.
