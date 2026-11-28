```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or say there is none.

**Per behaviour, the test that pins it**

1. `--gate batch` runs a tier above tier0 that does not cover tier0 over the cards' packages alone: the new row "--gate batch runs tier1 over the cards' packages alone…" (`scopes_integration_test.go:537`). It asserts `./lib` and `.` for tier1 at line 564.
2. Tier0 under `--gate batch` keeps the dependents: the same row, which asserts `scoped` for tier0 at line 564.
3. A batch tier that covers tier0 keeps the dependents: the same row, `writeCardKeys(true)` at lines 270–271.
4. The `card` key is not narrowed at any tier: the same row, the `--gate card` call at line 268.
5. The notice line naming the narrowed tiers: the same row, the `strings.Contains` check at line 265.
6. The guard step takes a dependent's tier1 guard when tier1's run is narrowed: the same row, which expects `./consumer -run ^(TestConsumerGuard)$` at line 264.
7. The guard step does not take a dependent's tier0 guard, because tier0's run holds the dependents: the same row, where `TestConsumerTierZeroGuard` is absent from `wantArgs` at line 264.
8. `listedPackages` replacing `listedTiers` for the plain `--card` form, where a covered or listed tier leaves out a scope package's guard: the existing row "--card runs a dependent package's guard in the guard step unless a listed tier is or covers its tier" (`scopes_integration_test.go:515`). The existing computed-scope rows cover the guard scan of the other forms.
9. The guard step does not repeat a tier1 guard test that sits in a card's own package when tier1's run is narrowed: none.
10. The main packages join the narrowed tiers' set under `--main-packages`: none. The `--main-packages` rows (`scopes_integration_test.go:835–865`) carry no `--gate batch`.
11. The narrowed tier's `//lym:watch` skip comes from the own packages (`ownSkip`): none. The only watch row for the card form, line 718, runs without `--gate batch`. In the new row no watched test sits in a dependent, so `sc.skip` and `ownSkip` are equal there.
12. No `ownSkip` when the changes cannot be read: none. This is the same gap as item 11.
13. The `existingDirs`/`watchedSkip` refactor of the existing watch scan: the existing row at `scopes_integration_test.go:718`. No row covers a card that names a package it has yet to create, but that gap predates the diff.
14. `gateslot.BatchGateKey` in `decodeGates`: the existing row `config_test.go:328`, "scope card on a key but card and batch".

**Findings**

- **MEDIUM**, class `design`, at `internal/gatecli/scope.go:248`. No test covers the main packages joining a narrowed batch tier under `--main-packages`.
  - If `own = withMainPackages(own, mainsDir, mains)` were dropped, the last batch's tier1 run would silently stop running the main packages, and no row would fail.
  - Evidence: `scopes_integration_test.go:835–865`, where every `--main-packages` row lacks `--gate batch`; the new row at lines 237–272 passes no `--main-packages`.
  - Fix: add a `--gate batch --card 1 --main-packages` call to the new row and expect the main package in tier1's invocation.
- **LOW**, class `design`, at `internal/gatecli/scope.go:257–266`. No test covers the narrowed tier's `//lym:watch` skip being computed over the own packages (`ownSkip`) rather than over `sc.skip`.
  - Using `sc.skip` would make the narrowed tier name, in its skip line and `-skip`, watched tests from dependent packages it does not run. No row exercises that, because no fixture puts a watched test in a dependent under `--gate batch`.
  - Evidence: `scopes_integration_test.go:718` (no `--gate batch`) and the new row at 237–272, which has no watched dependent.
- **LOW**, class `design`, at `internal/gatecli/scope.go:361–384`. No test covers a tier1 `//lym:guard` test in a card's own package being left out of the guard step when tier1 is narrowed.
  - If `listedPackages` dropped the own share for a narrowed tier, that guard would run twice, and no row would fail.
  - Evidence: the new row, whose fixture holds tier1 guards only in `consumer`, a dependent (lines 240–243).

**Tool calls**

1. One grep and read: the existing `--gate batch` rows, the config-key test for the batch scope check, any test of a card naming a package not yet created, and the watch-skip row at `scopes_integration_test.go:718–760`. My context held none of these.
