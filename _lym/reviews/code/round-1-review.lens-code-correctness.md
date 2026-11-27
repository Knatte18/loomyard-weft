```yaml
lens: code-correctness
outcome: reported
```

**Cases run**

- `markedScope`, scoper `none` → refused at only.go:89 ("needs the go scoper"). Correct.
- `markedScope`, `request.marked` empty, no `go.mod` → config refusal at only.go:98. Correct.
- `markedScope`, `request.marked` empty, `go.mod` present → "no test … is marked" refusal at only.go:102. Correct.
- `markedScope`, marked {tier0: heldpkg, tier1: heldpkg}, key lists [0] → returns guards for both tiers, no refusal. Correct per the record.
- `planTierRuns`, same input → listed tier0 run gets no invocations (tiers.go `scopeInvocations` `scopeMarked`); `ranWhole(runs, 1, sc)` false, so a tier1 guard run is planned with tier1's own command, slots and dir (`sc.guardDir`). Correct.
- `planTierRuns`, key lists [1] with tier1 covers 0, marked tier0 test → `ranWhole` false since the marked share is not whole, so the tier0 guard runs. Correct (unchanged).
- `planTierRuns`, marked test of undeclared tier5 → refused at tiers.go:230 via `configRefusal`, before any slot. Behaviour correct; wording NIT below.
- `planTierRuns`, marked test of a declared tier whose command lacks `{targets}`/`{args}` → refused by the guard-command refusal. Correct (previously unreachable unless listed).
- `runGuard` → `guardStepRun(tier1)` → `cfg.Tier(1)` ok; green entry recorded under tier1's fingerprint with `Packages` (green.go:462); run records carry `TierList` [0] like any other guard step of an unlisted tier. Correct.
- `notRunTiers`/`otherTiers`: a tier run only via the guard step still counts as "not run" for the dotnet filtered-notice; that notice is unreachable under the go scoper, which the marked scope requires. Correct.
- only.go imports after removing `markedNames`: `maps`, `slices`, `sort`, `strings` all still used. Correct.

**Findings**

1. MEDIUM, design — `internal/gatecli/only.go:88` (`markedScope`): the removed check was also the only thing that made a hub opt in before a **manual** tier's marked tests ran. A `//lym:only upkeep` test in a file tagged for a `manual: true` tier (one "that spends tokens", gateslot/tiers.go:50) now runs on every `--gate upkeep` call without anyone naming that tier; before, the key had to list it by number. Nothing in `planTierRuns`' guard loop (tiers.go:224–249) checks `Manual`, and guard tests could not reach it from a tier2/tier3 file (gatecli/cli.go:275 refusal), so the gap opens only for marked tests. Evidence: removed hunk (diff 001 only.go:-37..-49), gateslot/tiers.go:50–51, gatecli/tiers.go:224–249.
2. NIT, consistency — `internal/gatecli/tiers.go:230`: for a marked test of an undeclared tier (newly routed here by this diff) the message says "the guard tests of tierN run under its command"; the tests are marked tests, not guard tests, so the refusal misnames what it refused. Evidence: tiers.go:230 (planTierRuns T+94:136), only.go:105.

**Tool calls**

1. Grep `Manual` in gateslot/gatecli tiers.go plus impactset `lym:only` tier handling, and read green.go:455–470 — context lacked manual-tier semantics and the guard-run green-record branch.
