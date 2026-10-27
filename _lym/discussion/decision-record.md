# Decision record — darn-drift-await-row

## Goal

Darn drives the baseline capture the way loom does: Drift-Baseline starts a detached capture and a Drift-Await row holds the run until the capture passed or the parent deferred it, so the `wait` capture mode and the no-await-row branch go from the code.

## Scope

- `contracts/recipes/darn-recipe.yaml`: Drift-Baseline drops `capture: wait` and hands off to a new Drift-Await row, which carries `entry_gate: drift.enabled` and hands off to Darn-Write at once.
- `internal/shedrecipe`: the DriftBaseline entry stops accepting a `capture` key, and its doc drops the key.
- `internal/driftengine`: `Row.WaitCapture`, `Deps.AwaitRow`, `VerbStatus.AwaitRow` and the `awaitRow` parameter of `CountedCapture` go, with every branch only they reach; a counted capture is always a passed record with `awaited_at` set.
- `internal/loomcli` and `internal/driftcli`: the wiring stops computing and passing the await-row flag.
- Tests that pin darn's row list, edges, row indexes or the removed modes follow the new recipe.
- Docs: `internal/driftengine/doc.go`, `internal/shedrecipe/doc.go`, the recipe's own header comment and the darn paragraph of `docs/overview.md`.

## Decisions

### Drift-Baseline always detaches

Decision: Drift-Baseline starts the detached capture and, when Windows refuses the detached start breakaway, runs the capture in its own process, as loom's row does today; a held capture lock is left running.
Rationale: the board entry removes the `wait` mode; the breakaway fallback is no `wait` mode and loom still needs it.

### `RunCapture` and `Sleep` stay

Decision: `Deps.RunCapture` stays for the breakaway fallback, and `Deps.Sleep` stays for Drift-Await's wait; Drift-Baseline no longer reads `Sleep`.
Rationale: both have callers after the change, and the board entry removes only the code `WaitCapture` alone uses.

### `CountedCapture` loses its `awaitRow` parameter

Decision: `CountedCapture(scope, capture, found)` counts a passed record only with `awaited_at` set, and `Deps.AwaitRow` and `VerbStatus.AwaitRow` go with it.
Rationale: both recipes with Drift rows now have an await row, so no caller needs the other branch.

### Darn's Drift-Await reads no subject

Decision: darn's Drift-Await row takes no config key, the same as loom's.
Rationale: the DriftAwait entry rejects every key and spawns no agent.

### No transition code

Decision: a darn run parked between its drift rows when the new `lym` takes over gets no migration, and the recipe's header says such a run is settled by hand.
Rationale: the board entry's 2026-10-27 operator decision.

## Constraints

- `PATTERN-shed-recipe-registry`: Drift-Await is added as a recipe row keyed by the existing `DriftAwait` constructor, with its on/off switch in `entry_gate`.
- `PATTERN-refusal-way-forward`: the DriftBaseline entry's refusal of an unknown key keeps naming its way forward.
- `PATTERN-test-speed`: tests run through `lym gate test`, and a removed mode's test rows go rather than being kept for dead code.
- Docs land in the same commit as the code, per `CLAUDE.md`.

## Auto-mode assumptions

- `lym drift defer` already admits a blocked Drift-Await on any recipe, so darn needs no verb change for the board entry's defer bullet.
- `CheckMainCapture` already admits a blocked Drift-Await, so `lym drift capture` works at darn's new halt unchanged.
- The run report's phase map, which puts Drift-Await in the plan phase, needs no darn-specific change.

## Open risks

- A darn run parked on Drift-Baseline mid-capture when the new binary takes over finds the lock held and moves on without waiting; the orch settles it by hand, per the board entry.

## Acceptance criteria

- Darn's recipe builds with Preflight, Drift-Baseline, Drift-Await, Darn-Write in that order, Drift-Await carrying `entry_gate: drift.enabled`.
- A DriftBaseline row with a `capture` key is refused naming the way forward.
- A Drift-Baseline pass with the capture lock held returns Done without waiting, and one that starts a capture returns once the detached child's record exists.
- `CountedCapture` refuses a passed record without `awaited_at` and counts one with it.
- `lym drift waive` and Drift-Verify judge a counted capture by `awaited_at` on every recipe.
- Every Drift row of darn's recipe is skipped under the template's `drift.enabled`.
