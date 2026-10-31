```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out, F1 to F4.

**Findings**

1. **MEDIUM**, `internal/adoptcli/deps_test.go` `TestAdoptGateTiers` (F1).
   - **Summary:** The F1 proposal leaves its stated goal unmet: removing the `start.Tiers = adoptGateTiers(prime)` line in `hubDepsAs`'s `StartSuite` (`deps.go:158`) fails no test.
   - **Evidence:**
     - The review's `fix` key says the test must make "removing the Tiers assignment in hubDepsAs's StartSuite ... fail a row".
     - `TestAdoptGateTiers` calls `adoptGateTiers` directly, so it never reaches that assignment.
     - `TestAdopt` feeds `Tiers` through its own fake `StartSuite`.
     - `TestEnsureCheck` calls `ensureCheck` alone and never sets `Tiers`.
     - A real-hub path through `StartSuite` is unavailable in tier0. `startSuite` returns an error under `testing.Testing()`, so `ensureCheck` fails before `Tiers` is set.
   - **Counter-case:** Delete line 158 and every adoptcli tier0 test still passes.
   - **Possible change:** Pin the wiring in a tier1 test. Seed a `running` check record for the test's own pid in `fabricengine.AdoptScratchDir(h.Path)` and write a gate config with an adopt key. Then call `AdoptAs` and assert `Suite.Tiers`.

2. **LOW**, same test (F1).
   - **Summary:** The reviewer's "gate config without the adopt key returns nil" case is unreachable through `LoadConfig`, and the fixer's replacement does not cover the same path.
   - **Evidence:**
     - The fixer's first run gave `adoptGateTiers() = [0]; want []` for a `gates` map with only `push`. The template supplies a default `adopt` key.
     - The replacement row, `AdoptKeyNamingAnUndeclaredTierReturnsNone`, exercises `TierList`'s refusal for an undeclared tier instead.
     - No row covers a config that loads with no adopt key. That row would return the template's default tiers. The `adoptGateTiers` doc comment says "or holds no adopt key", which the code can never produce on a loaded config.
   - **Possible change:** Reword the comment, or leave it, since the substitute row covers the nil-on-`TierList`-error branch.

**Checked with no finding**
- **F2:** The fixture clock reads at reads 0, 1 and 2, giving the check start, tier0 start and tier1 start. A mark timed from the check's start would produce `checkFixtureAfter(0)` and fail the rows. The test comment states the coupling to the read order.
- **F3:** The `adoptFields` extraction is a sound variant of the proposal. Dropping the `fields["message"]` assignment now fails `TestAdoptFields`.
- **F4:**
  - The `\d\d:\d\d ` regex cannot match inside the mark text or the `bg=colour236` style strings, so the last match is the `%H:%M ` clock.
  - A mark rendered after the clock would fail.
  - The test is tier2, which the fixer could not run, so it has only been checked by reading.

**Further tool calls:** none.
