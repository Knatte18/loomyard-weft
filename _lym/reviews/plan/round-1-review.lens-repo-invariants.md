```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. I checked each card for invariants it breaks outside the lines it touches. Card 3 breaks a test it does not list.

**Per card: each invariant checked, held or broken**

- **Card 1 (owner-replace-set)**
  - Reflective seam over `OwnerRequest`: held. No reflective test walks it; `internal/beamcli/create_test.go:81` and `:415` fake `SetOwners` by func value.
  - Tier tag: held. The rows go into `internal/boardengine/ownerset_integration_test.go`, whose functions are all tier1 (`:158`).
  - `pattern/PATTERN-owner-machine.md` adopt clause (`:27`): held. The card edits it.
  - `pattern/PATTERN-owner-machine.md` Rule line (`:7`, "set only by `boardengine`'s setter"): held.
  - `internal/boardengine/doc.go` Owners section: partly broken. The card lists `doc.go`, but its Intent rewrites only the Takeover sentence. The commit-subject and "each named beam project" lines are left out (finding 3).
  - Card order: held. It is the first card.
- **Card 2 (hostname-name)**
  - Leaf import allowlist (`internal/machinename/machinename_test.go:231`): held. Only `os` is added.
  - Sole-source hostname scan: held. It reads production files only.
  - `internal/machinename/doc.go`: held. The card extends it.
  - Tier: held. tier0 joins `machinename_test.go`, whose other test functions are tier0 by the same convention.
- **Card 3 (machine-name-show)**
  - Registration oracle (`cmd/lym/registration_test.go:131`): held. `newRoot` is edited.
  - Root `Long` module list (`cmd/lym/clitree_test.go:152`): held. The card adds `machine`.
  - Sandbox coverage guard (`cmd/lym/sandbox_coverage_test.go:76-127`): broken (finding 1).
  - Seam signature lists (`cmd/lym/seamsignature_test.go:31-77`): not enforced, and the comment there goes stale (finding 4).
  - `testmain_test.go` under `tmuxkit.Main` (`PATTERN-test-isolation`): held.
  - `pattern/PATTERN-cli-cobra.md` naming deviation: held. The card extends it.
  - Card order: held. It uses `HostnameName` from card 2.
- **Card 4 (machine-name-set)**
  - One tier per test file (`cmd/lym/tiertags_test.go:2`): at risk. The card mixes tier0 and tier1 tests and names no file to split them (finding 2).
  - `internal/machinename/doc.go` "only production reader of … the machine file": held. `machinecli` reads through `machinename.ReadFile`.
  - `internal/machinecli/doc.go`: held. The card edits it.
  - Card order: held. `ReplaceOwners` comes from card 1, the leaf from card 3.

**Findings**

1. **BLOCKING · consistency · card 3 (`03-machine-name-show.md`)**
   - Summary: card 3 mounts the new `machine` module in `newRoot`. It neither adds a `**Covers:** machine` scenario to a `tools/sandbox/*SUITE.md` file nor an `excludedModules` entry in `cmd/lym/sandbox_coverage_test.go`.
   - Effect: `TestSandboxCoverage_AllModulesCoveredOrExcluded` (tier0, in card 3's gate) fails with "module "machine" is registered in newRoot() but has no **Covers:** tag…". Card 3 cannot pass its own gate.
   - Fix: add `cmd/lym/sandbox_coverage_test.go` to card 3, with an `excludedModules` entry whose reason names `internal/machinecli`'s own seam tests. Alternatively, add a Covers scenario to a suite file.
   - Evidence: `cmd/lym/sandbox_coverage_test.go:76-127`, `cmd/lym/main.go:198-201`.
2. **LOW · design · card 4 (`04-machine-name-set.md`)**
   - Summary: the card plans `TestNextFile` and `TestSetName` (tier0) and `TestSetName_SweepsRealHubs` (tier1) but names no test files.
   - Constraint: the tier scan requires all test functions in one file to name one tier, so the tier1 test needs its own file, such as `setname_integration_test.go`.
   - Fix: state the split in the card.
   - Evidence: `cmd/lym/tiertags_test.go:2`.
3. **LOW · consistency · card 1 (`01-owner-replace-set.md`, `internal/boardengine/doc.go`)**
   - Summary: card 1's Intent changes the commit subject (`, replacing &lt;set&gt;`) and writes entries no slug names.
   - Stale text: `doc.go`'s Owners section says the write "writes one commit "board: owner &lt;machine&gt; for &lt;slugs&gt;"" and "sets each named beam project's owner" (`doc.go:174-175`). The card lists `doc.go` but its Intent rewrites only the Takeover sentence.
   - Fix: the card's doc sentence also rewrites lines 174-175 for the replace set.
4. **NIT · consistency · card 3 (`cmd/lym/seamsignature_test.go`)**
   - Summary: the file's comment says it pins "every module's RunCLI" and "every module's RunCLIIn whose RunCLIIn reads the cwd it is given".
   - Gap: `machinecli.RunCLIIn` reads cwd (card 4's `cwdHub`), but no card adds `machinecli` to either list, so the comment goes stale.
   - Fix: add `cmd/lym/seamsignature_test.go` to card 3 or card 4 and list both seams.
   - Evidence: `cmd/lym/seamsignature_test.go:1-3`, `:54-56`.

**Tool calls**

1. A grep for reflective tests and other `OwnerRequest` test users, plus the tier-tag rule in `cmd/lym/tiertags_test.go`. My context lacked any seam test over `OwnerRequest` and the per-file tier rule.
2. A read of the `cmd/lym/uncontainedwrite_test.go` header and the path-rule guards. My context lacked whether a guard covers the machine-file write; it does not, since the guard is scoped to `fabricengine`.
