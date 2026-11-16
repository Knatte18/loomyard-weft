```yaml
lens: affected-set
outcome: reported
```

Lens affected-set: for each output a card changes, I traced what reads or asserts it and which card covers that.

**Per change: what reads it, and which card covers it**

- **Card 3, `machine` module mounted on `newRoot`.** Readers:
  - `cmd/lym/registration_test.go`: covered, card 3 edits `newRoot`.
  - `cmd/lym/clitree_test.go:152`, which requires root `Long` to name every module: covered, card 3 edits `Long`.
  - `cmd/lym/sandbox_coverage_test.go:76-118`: **not covered**.
  - `cmd/lym/seamsignature_test.go`: **not covered**.
  - `docs/overview.md`'s package tree: **not covered**.
  - `pattern/PATTERN-cli-cobra.md` Package naming: covered by card 3's Extends.
- **Card 1, new `OwnerRequest.ReplaceOwners` field.** Its callers are the `OwnerRequest{` literals in beamcli, adoptcli, battencli, orchcli, hubgeom, fronttask and loomcli. They leave the field empty, so they need no change. No reflective seam test reads `OwnerRequest`. Held.
- **Card 1, the owner commit subject gains `replacing &lt;set&gt;`.** Readers:
  - `internal/boardengine/doc.go:174` states the subject as `"board: owner &lt;machine&gt; for &lt;slugs&gt;"`: **not covered**. Card 1's `doc.go` edit names only the Takeover / replace-set sentence.
  - `TestSetOwners`' `wantSubject` rows: covered, since existing rows keep their subjects.
  - No other production code parses the subject (my grep found none).
- **Card 1, `newOwnerFixture` gains `closed` and `kept`.** Reader: the `BeamAndOwnerLandInOneCommit` assertion `held == ["#7d2d6b"]` at `ownerset_integration_test.go:437`, which reads every open beam's colour. This is only implicitly covered: the card does not state that `kept` carries no colour.
- **Card 1, the `PATTERN-owner-machine` bullet.** `PATTERN.md` line 24 is covered and unchanged.
- **Card 2, `HostnameName`.** The sole-source scan (`machinename_test.go:234`) reads only production files outside the package. Held.
- **Card 4, the leaf's `Use` and `Args` widened.** `clitree_test.go`'s `Use` grammar admits `[&lt;name&gt;]`. Held.

**Findings**

1. **BLOCKING · consistency · card 3 (`03-machine-name-show.md`).** Mounting `machine` on `newRoot` without a `**Covers:** machine` tag in `tools/sandbox/*SUITE.md` or an `excludedModules` entry fails the tier0 `TestSandboxCoverage_AllModulesCoveredOrExcluded`. Evidence: `cmd/lym/sandbox_coverage_test.go:82-118`, which collects every `root.Commands()` name and errors on any that is neither covered nor excluded. Its watch list includes `cmd/lym/main.go` and `internal/*cli/*.go`, so card 3's own gate runs it. No card targets the file.
   - Fix: add `cmd/lym/sandbox_coverage_test.go` to card 3's Edit group with an `excludedModules` entry for `machine`. The reason: it writes the per-user machine file and pushes every live hub's Bolt, so it is exercised by `internal/machinecli`'s own seam and hubforge tests instead.
2. **MEDIUM · design · card 1 (`01-owner-replace-set.md`).** The commit subject change reaches only part of what reads it.
   - `internal/boardengine/doc.go:174` documents the subject as `"board: owner &lt;machine&gt; for &lt;slugs&gt;"`. Card 1's `doc.go` edit names only the Owners-section takeover sentence, so the documented format goes stale.
   - The card pins the subject only "after any named subjects" (`board: owner neve for prime, replacing box`). Card 4's call passes `ReplaceOwners` with no `Slugs` and no `Prime` (`04-machine-name-set.md` step 8). For that request shape, `ownerset.go:101-105` builds `"board: owner &lt;m&gt; for "` with an empty list, and the card leaves the subject unpinned.
   - Neither `TestSetOwners` row asserts a subject for a replace-only write: `ReplaceSetRewrites…` sets prime owner `box` but no `Prime: true` and no `wantSubject`.
   - Fix: card 1 states the replace-only subject (e.g. `board: owner neve replacing box`), lists the doc.go:174 sentence in its `doc.go` change, and gives the `ReplaceSetRewritesOwnersRecordedUnderAFormerName` row a `wantSubject`.
3. **LOW · consistency · card 3.** `cmd/lym/seamsignature_test.go` pins every module's `RunCLI` and `RunCLIIn`. Its header says "every module's", and its comment admits a module whose `RunCLIIn` reads its cwd, as machinecli's does for `cwdHub`. Card 3 creates both seams and adds machinecli to neither list.
   - Evidence: `cmd/lym/seamsignature_test.go:1-3, 31-77`.
   - Fix: add `cmd/lym/seamsignature_test.go` to card 3's Edit group with `machinecli.RunCLI` and `machinecli.RunCLIIn`.
4. **LOW · consistency · card 3.** `docs/overview.md`'s package tree lists each internal package with a description, including `internal/machinename` at line 362 and `internal/hibernatecli` at line 384. Card 3 creates `internal/machinecli`, and no card adds its line.
   - Fix: add `docs/overview.md` to card 3 as a Prosa target with the `internal/machinecli/` line.
5. **NIT · consistency · card 1.** The ImpactSummary says the fixture's two new entries are ones "no existing row asserts on". But `BeamAndOwnerLandInOneCommit` asserts the colours held by every other open beam (`ownerset_integration_test.go:437`, through `heldBeamColors` at `ownerset.go:311-319`). `kept` is an open beam, so it must carry no beam colour for that row to hold.
   - Fix: state in card 1's Intent that `kept` and `closed` record no beam colour.

Tool calls:
1. Grep for readers of the `board: owner ` subject, `OwnerRequest{` users, reflective seam tests and module lists in docs: the manifest held no callers outside boardengine.
2. Grep for cmd/lym and testkit tests that enumerate modules, and for docs that list modules: to find module-enumeration readers beyond the ones the handler read.
3. Read the reexec guard enumeration and `docs/overview.md`'s hibernate lines: to judge whether they enumerate modules.
4. Grep `docs/overview.md` for the machine packages and for a test guarding it: to judge whether the tree needs a machinecli line and whether any test enforces it.
5. Grep for a websterengine or planparser guard on `overview.md`: to confirm no test fails on a missing package line.
