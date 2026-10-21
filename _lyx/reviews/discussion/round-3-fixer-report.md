# Round 3 fixer-report — discussion

Every finding's premise held against the code; all six are fixed in `_lyx/discussion/decision-record.md`.

## Changes

- **F1** (fixed as proposed): D3's marker bullet scopes "every registered module's base file exists" to the walk's pass.
  The clone bullet states the clone's own precondition (the board held no base file before its pass), that the walk's precondition does not apply to it, and that a clone marker preceding the absent per-worktree defaults is intended, since the clone runs without `SeedDefaults` and the walk seeds them after the marker, sparse.
  The first acceptance criterion is unchanged.
- **F2** (fixed differently): D3's "Every other caller" bullet now says those callers strip no present base file, and states that a seed of an absent base from the prime's copy is sparse (template-differing units, former values filtered while the prime has no `sparse` marker) in every caller that seeds, and that this is a seed of an absent file, not a strip, so `PATTERN-sparse-hub-base` covers it.
  The criterion now says `lyx fabric reconcile` and `lyx config reconcile --apply` strip no present base file.
  Departure from the proposal: its list of seeding callers left out `lyx fabric reconcile`, which also seeds absent hub-wide bases with open maps or `SeedsFromPrime` from the prime (`internal/fabriccli/fabric.go:706` passes `primeBaseDir`; `internal/configsync/configsync.go:592`), so the list names it beside the clone.
- **F3** (fixed as proposed): D3's marker bullet and the criterion now say "a failed hub-wide step", meaning a failed hub-wide reconcile or board commit, which `hubreconcile` restores (`internal/hubreconcile/walk.go:219`).
  D3 adds that a later worktree-step failure leaves the committed board and marker in place and the next walk strips no base again and retries the worktrees.
- **F4** (fixed as proposed): D3's first bullet adds that a base the strip, or the removal of keys the template lacks, leaves with no unit is written as D2's header-only base and never deleted, with the reason (`reconcileSparseFile` removes an emptied file, `internal/configsync/configsync.go:466`; a deleted base withholds the marker and keeps the stamp stale).
  The second criterion adds "a base the strip empties being present and key-less".
- **F5** (fixed as proposed): D1's weakened-guards paragraph adds the driver drop from `sonnet[medium]` to `haiku[medium]`, its bound under `shed`'s `driver_kind: go`, and the pin-back command.
- **F6** (fixed as proposed): D4's key-move bullet states that an absent target, in a pass seeding from the prime, takes the sparse prime seed first and the moved units are laid over it, and that a pass not seeding from the prime starts it empty.
  The plan note on `moveKeys` names `loadTarget`, which today starts an absent target from the prime's full copy, falling back to the template.

## Self-check

- fix-consistency, LOW, the key-move criterion still said "only the units … moved" after F6's prime seed: fixed as proposed; the criterion adds "bar a move target's sparse prime seed in a pass that seeds from the prime".
- fix-consistency, LOW, D4 did not say which value wins when a prime-seeded target unit and a moved unit collide: fixed as proposed; D4 adds that a target unit differing from the template, a prime-seeded one included, is kept and the moved value dropped, as `takeKey` does today (`internal/configsync/movekeys.go:226`).
- fix-claims, MEDIUM, F5's bound "no run uses the llm driver unless a hub sets `driver_kind: llm`" ignored the module override, task run settings and `--driver`: fixed as proposed; the bound now names all four selectors.
- fix-proposal: no findings.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
