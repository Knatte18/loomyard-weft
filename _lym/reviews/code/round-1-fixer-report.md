---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The watched test sits in consumer/c0_test.go as proposed, but that file is committed on the prime and fast-forwarded into the pair: written into the pair's working tree, as the proposal's fixture is, it counts as changed and a watched test whose own file changed is never skipped (internal/impactset/watches.go:76, seen in a failed run of the row)."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

# Fixer report, code round 1

## Changes

- **F1:** `goReadGates`, `runplan.RenderBatchGate`, Weaver's batch gate calls and entry name, and `loadGateConfig` use `gateslot.BatchGateKey`; `gateKeyBatch` is deleted.
- **F2:** the `--gate batch` row of `TestGateTest_Scopes` adds `tool/main.go`, passes `--main-packages`, and asserts tier1's root run is `./lib ./tool`.
- **F3:** `ownPackageTiers` leaves out a `carry: false` tier, every tier of a `--no-carry` call and a command with no target placeholder. No test was added, since the finding is not a coverage gap, so no test fails if the condition is removed.
- **F4 (fixed differently):** the same row asserts that tier0 skips `TestConsumerWatched` and the narrowed tier1 does not. The proposed untracked fixture file was not used; see the frontmatter.
- **F5:** the verify-model paragraph of `contracts/specs/loom-plan-spec.md` no longer gives every batch tier the dependents.
- **F6:** the `--card` flag help states the batch narrowing.
- **F7:** `_lym/landing/summary.md` no longer names the decision record. It is not committed, as no git command is run against `_lym`.

Commits: F1 alone, F2 with F4 (shared test file), then F3, F5 and F6 each alone, and one for the self-check.
The edits were gated together before the per-finding commits, not one commit at a time.

Verification: `lym gate test --tier 1` over `gatecli`, `gateslot`, `weaverengine`, `runplan` and `webstercli` passed (0 failed, 160 skipped), and its log shows the changed row passed.
After the self-check edits, the same tier over `./internal/gatecli` passed.

## Self-check

All four forks returned a report.

- fix-code-correctness, LOW (`computeScope` doc states the old rule): fix as proposed.
- fix-consistency, LOW (F3's exception missing from sibling statements): fix with own change. `computeScope`, `doc.go`, the spec bullet and the landing summary now state it. The `--card` help and the `template.yaml` comment are left as they are: a tier that runs whole ignores every card scope, which neither line states for any other scope.
- fix-consistency, LOW (stale `BatchGateKey` bullet in the landing summary): fix as proposed.
- fix-proposal, LOW (F3's exception missing from sibling statements): duplicate of the first fix-consistency finding, same disposition.
- fix-affected-set, LOW (F3's exception missing from sibling statements): duplicate of the same finding, same disposition.

## Disputed

## Deferred

## Rulings
