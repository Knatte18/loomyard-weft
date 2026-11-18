```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked three things: reflective seam tests, tier tags on new tests, and doc text the card makes false. I also checked that each card can pass its own gate from the state the earlier cards leave.

**Per-card list**

- **Card 1** (lymcwd): no reflective seam test. Both tests carry a tier (tier0, tier1). The `PATTERN-cwd-resolution` background bullet "`ResolveWithin` accepts cwd … anywhere in the records sibling" goes stale until card 16 rewrites it. That is listed, so it holds. The card can pass its gate on its own state: held.
- **Card 2**: additive only; tests are tier1; nothing in `doc.go` becomes false. Held.
- **Card 3**: no reflective test reads the geometry members; test is tier1; nothing becomes false. Held.
- **Card 4**: `landingshed/doc.go:336-337` still reads true once the match moves, since it describes the wording, not where the helper lives. The test rows give their tiers. Held.
- **Card 5**: no seam marker on `CloneOptions` or `CloneResult`. Every new test names a tier. Gate order holds: it uses only cards 3 and 4. Held.
- **Card 6**: the `beforeWire` seam already exists; test is tier1. Held.
- **Card 7**: `TestRetryHeldIndexLock` is tier0 with an injected sleep; the other test is tier1. Held.
- **Card 8**: tier1. Held.
- **Card 9**: test seam is **broken** (finding RI-1). The new `Fabric.afterMergeStart` field is not read by any reflective test I found, so that part holds.
- **Card 10**: depends on RI-1, because `UnsettledMergeReadsRecordsUnsettled` and the `RecordsSettled` asserts need the field visible to tests. Otherwise held.
- **Card 11**: no reflective test over `statuscommit.Deps` or either `commitStatusDeps`. The ones I found reflect over landing deps (`loomcli/landingdeps_test.go:127`), which no card changes. The `docs/overview.md:339` "skip while mid-merge" text is covered by card 15. The statuscommit package comment is rewritten by card 11 itself. Held.
- **Card 12**: the new `remoteTeardownJob.DeleteViewRef` field is not read by any no-zero-field or reflective test I found. Held.
- **Card 13**: tier0 and tier1 rows given. Held.
- **Card 14**: no reflective test in runfsm over `Inputs` or `Verdict`. Held.
- **Cards 15 and 16**: prose only. Held.

**Findings**

- **RI-1**
  - Severity: MEDIUM. Class: consistency.
  - Location: card 9, `internal/fabricengine#mergeState`; the members it misses are `internal/fabricengine#MergeStateForTest` and `internal/fabricengine#TestMergeState_Record`.
  - Summary: Card 9 adds `mergeState.RecordsSettled`, but no card lists the test mirror `MergeStateForTest` or its convert functions `toMergeState` and `fromMergeState`.
    - `TestMergeState_Record` calls `assertNoZeroFields` on the mirror and claims it covers "every field". The new field would round-trip zero-to-zero, so the test stays green while what it claims goes unchecked.
    - Card 9's row `RecordsWriteBetweenStepTwoAndThreeRefuses` and card 10's unsettled rows assert `RecordsSettled`, but the external tests can only read it through `LoadMergeStateForTest`. Card 9 as written does not pass its own gate.
    - Fix: add the mirror and `TestMergeState_Record` (with `RecordsSettled: true` in `want`) to card 9's Edit list.
  - Evidence: `internal/fabricengine/export_test.go:526-543` (the mirror has no such field); `internal/fabricengine/mergestate_integration_test.go:51,79-97` (seam marker, `want`, and `assertNoZeroFields`); `internal/fabricengine/mergestate.go:47-61`.

**Tool calls**

1. Searched for `//lym:seam` markers over the structs the cards change: the reading manifest held no seam-marker inventory.
2. Read the seam test and the `MergeStateForTest` mirror: the only marker found pointed here, outside the manifest.
3. Read the `TestMergeState_Record` body, the mirror fields and `mergeState`: needed to confirm the "every field" check and that the mirror lacks the new field.
4. Searched docs, PATTERN files and `doc.go` files for skip and protected-branch wording, and for reflective tests in the seam packages: to find doc text the cards make false that the manifest didn't cover.
5. Read `landingshed/doc.go:330-342` and the teardown-job tests: to judge whether card 4 makes the doc false and whether card 12's new field hits a no-zero-field test.
6. Searched `Fabric.Commit` for its nil-file and code-side path: an order check for card 7's empty-commit row. It settled nothing, and no finding rests on it.
