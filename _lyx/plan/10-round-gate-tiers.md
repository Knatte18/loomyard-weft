# Card 10 — round-gate-tiers

**Create:**
- `plan:internal/verifytree#FailureKindLanding` -> `const FailureKindLanding FailureKind = "landing"`
- `plan:internal/verifytree#FailureKindPublish` -> `const FailureKindPublish FailureKind = "publish"`
- `plan:internal/loomshed#RoundGate` -> `type RoundGate struct`
- `plan:internal/testkit/lyxbin#PutFirstOnPath` -> `func PutFirstOnPath(t *testing.T)`

**Edit:**
- `internal/verifytree#PublishFailure`
- `internal/verifytree#TestPublishFailureRecord`
- `internal/verifytree/doc.go`
- `internal/loomshed#NewVerifyGate` -> `func NewVerifyGate(worktreeRoot, verifyDir, siteLabel string, command, mergeBase func() (string, error), treeScope func() (string, bool, error), slots *gateslot.Pool, round func() (RoundGate, error)) shuttleengine.Gate`
- `internal/loomshed#checkedPublishFailure`
- `internal/loomshed#publishFailureSteps` -> `func publishFailureSteps(failure verifytree.PublishFailure, tiers func(gate string) ([]int, error)) (failedTests, packagePass string, err error)`
- `internal/loomshed#PublishFailureNote`
- `internal/loomshed#TestVerifyGate_Scenario`
- `internal/loomshed#TestVerifyGate_RoundScenario`
- `internal/loomshed#roundGateFixture.runGate`
- `internal/loomshed#roundGateFixture.diffGate`
- `internal/loomshed#TestPublishFailure`
- `internal/loomshed#TestPublishFailureNote`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/testkit/envkit#FullEnv`
- `internal/webstercli#TestGateParity_VerifyGate`
- `internal/impactset#TestDerive_OverARepository`
- `internal/impactset/doc.go`
- `contracts/specs/loom-status-spec.md`

**Delete:**
- `internal/loomshed#guardScanFindings`
- `internal/loomshed#putLyxFirstOnPath`
- `internal/impactset#Derive`
- `internal/impactset#Derivation`
- `internal/impactset#changedPathsSince`
- `internal/impactset#commandFor`
- `internal/impactset#buildCommand`
- `internal/impactset#vetTags`
- `internal/impactset#derive`
- `internal/impactset#gateCommand`
- `internal/impactset#TestCommandFor`
- `internal/impactset#TestCommandFor_OverLongCommandFallsBack`

**Uses:**
- `internal/testkit/lyxbin#Build`

**Intent:**
The Webster-Burler-Fix, Merge-Burler-Fix, Darn-Write and Darn-Burler-Fix gates run the hub's `round` gate through `lyx gate test --wait` instead of spelling an impacted-set command (D8).
The card comes right after the tier verb and its scopes (cards 8 and 9), before any later card targets `internal/loomshed`, because the round scenario test runs the derived command through the `lyx` built from the tree, which refuses `--tags` from card 8 on; no batch gate between card 8 and this card runs that test.

The told command is the site's base command, read at each arrival: loom's `final` call and darn's `landing` call, which the wiring renders (card 14), so `BaseCommand` keeps its record entry as today.
The new `round` seam returns a `RoundGate{Tiers []int, Render func(scope ...string) string}` read from the hub's `gate.yaml` at each arrival, so the gate renders `lyx gate test --wait --tier <round list> <scope>` without parsing config itself; a seam error is the gate's returned error, carrying the config refusal's own way forward (card 2).
- Round form: with a recorded pass of the base command, the gate runs the round call with `--base <that pass's commit>`; with none it runs the base command itself.
  The comment lint runs from that commit as today, and the headroom lint from the told merge base.
- Darn form: the lints run from the merge base as today; a tree-scoped diff runs the tree-scoped command, and any other diff the round call with `--base <merge base>`.
  A tree-scoped diff whose command is empty, as for a docs-only diff whose trees name no target (card 14), runs no command, logs that nothing ran and passes, so an empty tree scope never falls back to the computed scope.
The gate-side `impactset.Derive` call, its fallback logging and the guard-scan findings go: the gate's own computed scope narrows, falls back to whole and refuses a guard scan error with its file and line, which reach the findings through the failed run's log tail.
`impactset.Derive`, its `Derivation`, its command builder and their tests go with their last caller; `DeriveTargets` (card 5) stays the one derivation.
`TestDerive_OverARepository` keeps every row, each `Derive` row recast as a `DeriveTargets` row with the same fallback or package set.
`slots` stays on `NewVerifyGate` until card 12 drops it.

The Publish failure record names its failing tier (D8): `PublishFailure` gains `Tier int` (`tier`, always written, since `tier0` is a real tier), and the kinds become the gate names `landing` and `publish`.
`checkedPublishFailure` accepts the two new kinds and the two old ones, and maps an old record: `plan_verify` reads as the `landing` list and `publish_verify` as the `publish` list, its tier the highest of that list; a new record whose `tier` is not on its gate's list reads the same way, as the writer's fallback (card 14) does.
`publishFailureSteps` builds the confirm-the-fix calls: one `lyx gate test --wait --tier <failing tier> <package> -- -run '^<Test>$'` per failing top-level test, its package an explicit target, and one package pass `lyx gate test --wait --tier <failing tier> <packages>` over the record's failing tests' packages; they join the round call with `&&` as the raw `go test` steps did, the failed tests before it and the package pass after it, and every argument is still a checked identifier.
A failing tier whose command lacks `{targets}` or `{args}` gets the gate's own handling of such a tier: it runs whole, or refuses words after `--` naming the tier (card 8).
A failing test the fix renamed or removed discovers zero there and passes.
`PublishFailureNote` names the failing gate and tier.

`shedrecipe.Env` gains the `round` seam beside `VerifyCommand`, which the verify gate entry requires and passes to `NewVerifyGate`; `envkit.FullEnv` fills it with a fixed round list, and the wiring fills it in card 14.
`lyxbin.PutFirstOnPath` is loomshed's `putLyxFirstOnPath` moved into the kit, since card 13's webster tests need the same step (`PATTERN-testkit`).
`doc.go` of loomshed and impactset, and `contracts/specs/loom-status-spec.md`'s running-verify example, name the rendered gate call.

Tests: `TestVerifyGate_RoundScenario` is rewritten over a scratch module seeded with a worktree `gate.yaml` whose tiers run real `go test -json` through the built `lyx`, put first on `PATH` by `lyxbin.PutFirstOnPath`: the base command run with no recorded pass, the round call with `--base` after one, the comment lint from the pass's commit, the darn form's `--base <merge base>`, its tree-scoped command and its empty tree-scoped command passing with nothing run, the Publish failure record's failed-test calls before and package pass after the round call, a record whose failing test was renamed passing with zero discovered, and an old-kind record mapped to the `landing` list.
`TestVerifyGate_Scenario` gains the row of a `round` seam error returned as the gate's error; it, `roundGateFixture.runGate` and `roundGateFixture.diffGate` pass the round seam.
`TestPublishFailure` swaps its tag rows for tier rows, old-kind mapping rows, the off-list tier row and the refusal path; `TestPublishFailureNote` names the gate and tier.
`TestPublishFailureRecord` round-trips the `tier` field, `0` included.
`TestDerive_OverARepository` keeps its fallback rows (no `go.mod`, an empty base, a base off HEAD's ancestry, a `go.mod` change, a deleted file whose directory is no longer a package) as `DeriveTargets` rows.

**ImpactSummary:** the round gate loses its in-process derivation and gains one told seam; the failure record gains one field and two kinds, old records still read; impactset's command builder and its tests are deleted with their last caller, and one test helper moves into the lyxbin kit.
