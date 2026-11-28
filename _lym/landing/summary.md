# Batch gate runs tiers above tier0 over the cards' own packages

Since `a69580709` widened the `--card` scope to every package whose build or tests reach a card's packages,
the batch gate's tier1 run spends most of its time in those dependent packages.
`docs/research/webster/gate-time-2026-11-28.md` measured its listed tier1 run at 141.8 s per run
over 140.5 packages after that change, against 32.6 s over 12.4 packages before it,
and found no batch tier1 failure in a dependent package since.
The whole-tree gate at the end of a plan still runs tier1 over everything.

## What changes

- **`--gate batch` narrows its higher tiers.**
  Under the `go` scoper, each listed tier of the `batch` key above tier0 that does not cover tier0
  runs the cards' packages alone, plus the main packages when the call passes `--main-packages`.
  Tier0, and a listed tier that covers tier0, keep the dependents.
  A tier that runs whole (`carry: false`, `--no-carry`, or a command that takes no targets) is not narrowed.
  The call prints one stderr line naming the tiers it narrowed.
  The `card` key keeps the widened scope at every tier, and no other key changes.
- **The guard step follows each tier's own share.**
  `scanGuards` used to treat a guard test as covered by the listed runs
  whenever its package was in the card scope and its tier was listed or covered.
  A scope now records, per tier, the packages in which the listed runs run that tier's tests (`listedPackages`),
  so a dependent package's tier1 `//lym:guard` test still runs in a batch call's guard step.
  The `//lym:watch` skip of a narrowed tier is computed over that tier's own packages.
- `gateslot.BatchGateKey` names the `batch` key wherever the code reads that key.
- The `--card` flag help, the `gatecli` package doc, the `gate.yaml` template comment
  and the plan spec's verify-model text state the narrower tiers.

## Design decisions

- The narrowing is a rule of the `batch` key in code, not a new `gate.yaml` field.
  A field would need a hub config write before it took effect, and the change concerns this one key's behaviour.
- The rule covers every tier above tier0 rather than naming tier1.
- A tier that covers tier0 keeps the dependents,
  so that tier0's tests of dependent packages still run at the batch gate on a hub where tier1 stands in for tier0.
- The main packages stay in the narrowed tiers:
  the last batch's `--main-packages` call exists to catch `cmd/lym` failures before the plan-level run.
  Of the runs behind the 32.6 s mean above, only a plan's last batch could hold them.

## Expected effect

The research doc estimates the narrower tier1 run at 4.3 min per implementer row against 18.9 min today,
about 14.6 min saved per row.
That is an estimate from per-run times of earlier runs; no run of the new scope has been timed.
It counts the listed tier1 run alone.
The guard step now also runs the dependent packages' tier1 `//lym:guard` tests,
and that added time is not subtracted, so the net saving may be smaller.

## To check by hand

- Test coverage of the main packages joining a narrowed batch tier under `--main-packages`
  (`own = withMainPackages(...)` in `internal/gatecli/scope.go`).
  The `--gate batch` row of `TestGateTest_Scopes` in `internal/gatecli/scopes_integration_test.go`
  passes `--main-packages` and asserts that tier1's root run is `./lib ./tool`;
  check that this fails if that line is dropped.
- No test covers the exception for a tier that runs whole (`ownPackageTiers` in `internal/gatecli/scope.go`).
- A failure in a dependent package that only a narrowed tier catches
  now surfaces at the plan's whole-tree run instead of the batch gate.
  The research sample had none; check that a later, more costly catch is acceptable.

## Merge danger

**Door:** two-way
**Blast radius:** gate
