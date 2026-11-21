---
round: 2
focus:
  - "internal/gatecli/tierrun.go `runInvocations`: settle whether the `--results-directory`, `--logger trx;LogFilePrefix=gate` and `--blame` words reach only tiers that opt in, or every `counter: vstest` tier whose command holds `{args}`, and whether docs/dotnet-hub.md's timing tier (`dotnet run ... -- {args}`) keeps its meaning."
  - "internal/gateslot/tiers.go `Config.MeasureCall`: settle whether every call it renders is one `planTierRuns` and `gateKeyRefusal` in internal/gatecli accept, covering a hub with no tier0, a tier whose command holds no `{args}`, and a card-scope key; and whether the `--tier` fallback is consistent with pattern/PATTERN-test-speed.md and pattern/PATTERN-gate-slot.md."
  - "internal/gatecli/tierrun.go `envOverrideLine` and its callers: settle what the line tells a non-prime session, which values it prints to stderr and the tier log, and how often it prints under `--copies`."
  - "internal/gatecli/tierrun_test.go and the tests added for the TRX reading: settle whether each test's `tiergate.Require` tier matches what it spawns, whether any skips at run time by OS, and whether the tests pin the baseline across invocations, the outcome mapping, the console-count fallbacks and the words reaching the command."
  - "internal/gatecli/trx.go and `readTRXResults`/`replaceInvocation`: settle which run messages become failure causes, and whether the counter's pending unterminated console line is consumed before the TRX replacement."
---

Focus for round 2 after a `CONTINUE` verdict in round 1.
