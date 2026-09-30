---
round: 1
focus:
  - "Card 3's ImpactSummary names only `lyx config` surfaces. `configreg.Modules()` is also iterated by `internal/configsync`'s `ReconcileAll`, which `internal/fabriccli/clone.go` calls with apply=true, and the registry is consumed by `internal/fabriccli/cloneconfigcommit_integration_test.go`, `internal/hubforge/hub_test.go` and `internal/configcli`'s reconcile tests. Does that conclusion cover fabric clone now seeding and committing `logger.yaml`, and does any of those tests pin a module or file list that card 3 would have to edit?"
  - "If card 3 turns out to need `internal/configcli`, `internal/configsync` or `internal/fabriccli` as targets, the plan's `## verify:` section runs `-tags integration` only for logger, loggerconfig, configreg and cmd/lyx. Settle whether every package the cards actually need to target has its tagged tests run."
  - "Card 5 places the exit hook (`NotifyExitAndSweep`) in `internal/loggerconfig`, while decision record D4 says the CLI layer loads the config and names the site as `cmd/lyx` root or a cliwire helper. Is that placement within what D4 and D3's step 2 license, or does it add scope the decision record does not name?"
  - "Tests in cards 4 and 5 run under the `go test` gate, where `resolveSinkDirLocked` refuses the cwd-anchored path unless a `SetDurableSinkDir*` override is in force. Can card 4's 'After `NotifyExit(1)` with no prior record, it reports `Armed: true`' case, and card 5's exit_test cases, be written as specified, and does each card state the redirect that case needs?"
  - "Walk each acceptance criterion in the decision record against the card that tests it, especially 'sweeps its sink directory exactly once at exit (zero and non-zero exit codes)' and 'a process that never armed … loads no config at exit'. Settle whether some card's test list carries each one."
---

Seeded from a read of the five cards, the decision record, and the code they edit.
Card 3's blast radius is the most concrete lead:
`internal/fabriccli/clone.go:100` calls `configsync.ReconcileAll(res.WeftBase, true)`, which loops over `configreg.Modules()` (`internal/configsync/configsync.go:80`).
So the registration reaches past the `lyx config` surfaces the card names.

Checked and left out of focus:
the missing-key error in card 2 (`configengine.load` already rejects missing keys);
the `configStrictnessDegradingSet` edit (the guard scans for `configengine.LoadOrTemplate(` call sites, and card 2 adds exactly one);
the Warn line on stderr in card 5's integration test (the default stderr threshold is Warn);
and `ImpactSummary` on the Create-only content (the per-type table does not require it).
