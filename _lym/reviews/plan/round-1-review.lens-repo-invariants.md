```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked three kinds of invariant: test seams and test-file tiers, doc text the card makes false, and whether the card can pass its own gate from the state the earlier cards leave.

**Per-card invariant list**

- **Card 1 (gate-test-map-mode)**
  - Seams: held. No `//lym:seam` marker names `internal/gatecli` or `internal/testcover`, so the new `testRequest.mapMode` field needs no seam test.
  - New test files: none.
  - Doc text: mostly held.
    - The live `testmap` mentions in gatecli `doc.go`, gateslot `doc.go` and `template.yaml`, dobbyengine `doc.go`, `README.md`, `docs/overview.md`, `PATTERN-test-speed` and `PATTERN-daemon-wakeups` are covered.
    - `PATTERN.md` holds no `testmap` text.
    - `cmd/lym/daemonwakeups_test.go`'s `unboundedWaitFiles` entry is covered.
    - Broken: the `internal/gatecli/testmapstamp_test.go` file header (NIT below).
  - Card order: broken against `PATTERN-test-only-code` (MEDIUM below).
- **Card 2 (testcover-planning-delete)**
  - Seams: none.
  - Doc text: partly broken. Its Intent misses the `GoRoot` sentence in testcover `doc.go` (LOW below).
  - Card order: held. After card 1 nothing outside testcover references the deleted symbols. `declcompare.go`'s helpers are used only by `testselect.go`, and the remaining `testselect.go` helpers are used only inside that file.
- **Card 3 (map-run-tier-bounds)**
  - Seams: the new `testMapRequest.guardFor` field needs no seam test (no marker exists).
  - New test functions: tier stated but their file is not (LOW below).
  - Doc text: held. The `prebuild.go` header, gatecli `doc.go`, `docs/overview.md`, gateslot `doc.go`, `PATTERN-gate-slot` and `PATTERN-test-speed` are covered.
  - Card order: held. Nothing card 3 deletes is reached afterwards. `prebuildLym` keeps its tagged-tier caller.
- **Card 4 (map-run-clean-checks)**
  - Seams: none.
  - Doc text: held. Card 4 covers gatecli `doc.go`, `docs/overview.md` and `PATTERN-test-speed`. Card 1 already rewrites the `testmaptree.go` header.
  - Card order: held. `readHead`, `dirtyOwnedFile` and `readPackageTree` survive cards 1–3. `changedFiles` and `changedName` stay in use until card 4 deletes them with their caller `endJob`. `emptied` comes from card 1's subtest.
- **Card 5 (redundant-map-way-forward)**
  - Seams: none.
  - Doc text: held. Card 1 already rewrites the gatecli `doc.go` lines naming the verb.
  - Card order: held.

**Findings**

1. **MEDIUM, design. Card 1, with card 2: card 1 leaves testcover's planning code unreachable, which fails the test-only-code guard until card 2 lands.**
   - Card 1 deletes the last production callers of `testcover.PlanUpkeep`, `NewSelector`/`Selector`, `Map.FailedAt` and `Toolchain.GoRoot` (`testmapplan.go:68,88,133,141`; `testmapaudit.go:37`). Card 2 deletes the symbols themselves.
   - Evidence: `cmd/lym/testonlycode_linux_test.go:79,81`, a `//lym:guard` tier1 test.
   - The guard step runs guard tests whatever the scope, and the `batch` gate runs tiers 0 and 1. So any batch gate that runs after card 1 and before card 2 fails on every newly unreachable testcover function.
   - Nothing in the plan forces cards 1 and 2 into one batch.
   - Fix: fold card 2's deletes into card 1, or have card 1 add allowlist entries that card 2 removes.

2. **LOW, consistency. Card 3: the file for `TestMapSideBySide` and `TestMapSlotAsk` is not named.**
   - Card 3 declares them as tier0 tests through `plan:internal/gatecli#…` handles with no file.
   - `PATTERN-test-speed.md:24` requires the test functions of one file to name one tier, and it is enforced by `cmd/lym/tiertags_test.go`.
   - The obvious home, `testmap_integration_test.go`, is tier1 (`testmap_integration_test.go:4,715`), so placing them there breaks the rule.
   - Fix: name a tier0 file, such as `testmapstamp_test.go`, or a new `Create` file with its tier.

3. **LOW, consistency. Card 2: testcover `doc.go` keeps describing `Toolchain.GoRoot`.**
   - Card 2 deletes `Toolchain.GoRoot` (`runner.go:132-134`).
   - Its `doc.go` edit names only the `PlanUpkeep` and `Selector` paragraphs and the `Map.FailedAt` sentence.
   - The sentence "and GoRoot its root directory" at `internal/testcover/doc.go:68` stays, describing a method that no longer exists.

4. **NIT, consistency. Card 1: the `testmapstamp_test.go` header goes stale.**
   - Card 1 deletes the plan's selector and rewrites `stampEnvironment`'s comment to drop it.
   - The test file header `internal/gatecli/testmapstamp_test.go:1` still says the stamp is one "a coverage map build and the plan's selector both compute".
   - No card lists that file.

Tool calls:
1. One grep of `cmd/lym/testonlycode_linux_test.go` for its tier and guard marker, of every test for `//lym:seam` markers on gatecli or testcover, of `PATTERN.md` for `testmap`, and of other tests naming the verb. The manifest held none of these.
