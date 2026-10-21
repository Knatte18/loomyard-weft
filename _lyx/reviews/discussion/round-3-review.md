---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:87
    summary: "D3 withholds the marker until every registered module's base file exists, but the clone's own reconcile pass leaves every per-worktree repo default absent, so the clone cannot write the marker D3 and the first acceptance criterion require"
    fix: >-
      In D3, scope the precondition in the marker bullet ("only when every registered module's base file exists after the pass") to the walk,
      and give the clone its own precondition in the clone bullet: the board held no base file before the clone's pass, and the walk seeds the absent per-worktree repo defaults after the marker, sparse (D4).
      Add one clause stating the consequence, that a marker the clone writes before those defaults exist is intended, since the clone's pass runs without SeedDefaults and leaves them absent.
      Leave the first acceptance criterion as it is.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:108
    summary: "D4's prime-seed bullet drops template-equal and former-value units for callers other than the walk, against D3's rule that every other caller strips nothing and the criterion that lyx config reconcile --apply strips nothing"
    fix: >-
      In D3's "Every other caller" bullet, add that a seed of an absent base from the prime's copy writes only the units that differ from the template, and the former-value filter while the prime has no sparse marker, for every caller that seeds:
      the walk, lyx config reconcile --apply --layer default (it passes SeedDefaults) and lyx fabric clone (its hub-wide modules with open maps or SeedsFromPrime seed from the weft base).
      State that this is the sparse seed of an absent file and not a strip of a present one, so PATTERN-sparse-hub-base's "only the units the hub tuned" covers it.
      Reword the criterion "lyx fabric reconcile and lyx config reconcile --apply on an unmigrated board strip nothing" to say they strip no present base file.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:177
    summary: "The failed-walk criterion and D3's removal of the marker on a failed walk cover more than the code can undo, since the hub-wide board commit lands and is pushed before the worktree steps run"
    fix: >-
      In D3's marker bullet and in the acceptance criterion "A failed walk leaves the board's config files at their prior bytes and no marker the pass wrote",
      replace "a failed walk" with "a failed hub-wide step", meaning a failed hub-wide reconcile or board commit, which hubreconcile already answers by restoring the prior bytes.
      Add that a failure in a later worktree step leaves the committed board and its marker in place and the next walk retries only the worktrees, which D3's worktree former-value drop already allows with the marker present.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:82
    summary: "D3's base strip does not say that a base it empties stays present as the header-only base, although the marker needs every base present and the worktree strip it parallels deletes an emptied file"
    fix: >-
      In D3's first bullet, add that a base file the strip, or the reconcile's removal of retired keys, leaves with no unit is written as D2's header-only base and never deleted.
      The sparse-file reconcile that the worktree strip uses removes an emptied file, and a deleted base would withhold the marker and keep the build stamp stale on every start verb.
      Add "a base the strip empties is present and key-less" to the second acceptance criterion's "holding no unit".
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:56
    summary: "D1's list of weakened guards leaves out the driver model, which D1 lowers from sonnet[medium] to haiku[medium], so the Bound does not name every model downgrade"
    fix: >-
      In D1's paragraph that lists the weakened guards, add that the llm-driven run's driver drops from sonnet[medium] to haiku[medium].
      Add its bound: shed's driver_kind: go means no run uses the llm driver unless a hub sets driver_kind: llm, and a hub pins the model back with lyx config loom --layer default --set driver=sonnet[medium].
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:110
    summary: "D4 starts an absent move target empty, so the move writes the file first and the module's own prime seed in the same pass sees a present file and never runs, losing the prime's tuned units"
    fix: >-
      In D4's key-move bullet, state the order for a target that is absent when a move reaches it in a pass that seeds from the prime:
      the target takes the sparse prime seed first (the first two seeding bullets), then the moved units are laid over it.
      A pass that does not seed from the prime starts the target empty, as written.
      Name moveKeys' loadTarget, which starts such a target from the prime's copy today, in the plan note on moveKeys.
---

### [MEDIUM] The clone's marker contradicts the marker's precondition

**Location:** `_lyx/discussion/decision-record.md:87`, `:90`; `internal/fabriccli/clone.go:57`; `internal/configsync/configsync.go:579`
**Issue:** D3 writes the marker "only when every registered module's base file exists after the pass", and also has `lyx fabric clone` write it in its own board commit.
The clone's `ReconcileHubWideAt` call passes `HubWideOptions{Apply: true}` with no `SeedDefaults`, and `ReconcileHubWideAt` leaves an absent per-worktree repo default absent in that case (`absentUnseeded`).
So after the clone's pass the precondition is false for every per-worktree module, and the clone writes no marker.
The first acceptance criterion ("its marker is committed by the clone") then fails, and a `--set` made between the clone and its walk is stripped by the walk, which is the hazard the clone's marker exists to prevent.
D4's own text assumes the other order, since it says the walk seeds the repo defaults "after the clone wrote the board marker".

### [MEDIUM] A prime seed drops units for callers D3 says strip nothing

**Location:** `_lyx/discussion/decision-record.md:89`, `:108`, `:176`; `internal/configcli/hubwide.go:180`; `internal/fabriccli/clone.go:57`
**Issue:** D4 seeds a base from the prime's copy with only the units that differ from the template and, while the prime has no `sparse` marker, from the former values.
That seed runs in every caller that seeds: the walk, `lyx config reconcile --apply --layer default` (`SeedDefaults: true`) and `lyx fabric clone` (a hub-wide module with open maps or `SeedsFromPrime` reads the weft base).
D3 says every caller but the walk "strips nothing", and the criterion says `lyx config reconcile --apply` on an unmigrated board strips nothing, so the text disagrees with itself about which callers drop template-equal and former-value units.
The seed of an absent file is what the declared `PATTERN-sparse-hub-base` text asks for ("only the units the hub tuned"), so the entry is not violated; the record's three statements need to agree.

### [LOW] The failed-walk criterion exceeds what the code can undo

**Location:** `_lyx/discussion/decision-record.md:88`, `:177`; `internal/hubreconcile/walk.go:60`, `:199`
**Issue:** `reconcileHubWide` commits and pushes the board before `walkHub` reconciles the prime and the pairs.
A failure in a later worktree step returns an error with the board commit, and so the marker, already landed; nothing restores the board's files at that point.
The criterion "A failed walk leaves the board's config files at their prior bytes and no marker the pass wrote" holds only for a failure inside the hub-wide step, where `failRestoringConfig` restores the snapshot.

### [LOW] An emptied base is not stated to stay present

**Location:** `_lyx/discussion/decision-record.md:82`, `:87`, `:170`; `internal/configsync/configsync.go:466`
**Issue:** D4 states for key moves, source and target, that an emptied base stays as the header-only base, and D2 states what a key-less base is.
D3's strip is a third writer that can empty a base, as every unit of a freshly seeded old base equals the template, and the worktree strip it parallels (`reconcileSparseFile`) deletes a file it empties.
The marker needs every base present, so a deleted base leaves the marker unwritten and the stamp stale until a later walk reseeds it.

### [LOW] D1's Bound omits the driver model downgrade

**Location:** `_lyx/discussion/decision-record.md:40`, `:50`
**Issue:** D1 lists `driver: haiku[medium]`, and the loom template holds `driver: sonnet[medium]`.
The paragraph that names the weakened guards lists the review budgets, the Webster-Review accept rule, and the reviewer and fixer downgrades, and not this one.
The driver decides stops of an llm-driven run, so it is a model downgrade of the kind the paragraph exists to list.
The Bound also needs to say that `shed`'s `driver_kind: go` makes the driver model unused unless a hub opts into `llm`.

### [LOW] A move into an absent target skips the prime seed

**Location:** `_lyx/discussion/decision-record.md:108`, `:110`, `:192`; `internal/configsync/movekeys.go:96`, `internal/configsync/configsync.go:592`
**Issue:** `moveKeys` runs before the per-module loop in `ReconcileHubWideAt`.
Today `loadTarget` starts an absent target from the prime's copy when the pass seeds from the prime, and the loop then sees a present file.
D4 says a move never starts an absent target from the prime's full copy, so the move writes only the moved units, the loop sees a present file, and the prime-seed bullet never runs for that module.
The prime's tuned units of a module a move targets are then never seeded.
The case needs a target module with no board file but a tuned prime copy, so it is rare.

## Rejected

- The record's treatment of the older-binary refill is complete against the code: it names the old walk, `lyx fabric reconcile`, `lyx config reconcile --apply --layer default`, `--set` on the base and `lyx fabric clone` of an existing weft, and states the repair and its limit; the only other `ReconcileHubWideAt` caller is the walk's own.
- D4 states that both an emptied source base and an emptied target base stay present as the header-only base, and the key-move acceptance criterion covers the source; the target case cannot arise, since `takeKey` always leaves the target holding the moved or the kept unit, which is a surplus clause and not a gap.
- D1's Webster-Review accept rule matches the code: only the Merge-Review Bouncer sets `blocking_escalates`, and the Webster-Review Bouncer carries its open findings into `## Open risks` through `carry_over`; the record's bound is adequate for the task's own change, which runs under loomyard's hub values already.
- D3's worktree former-value drop is covered: the unmarked worktree unit that equals a former value while the base holds another tuned value is stated in the auto-mode assumptions, and the walk, the clone's walk, the mid-merge pair's next walk and `Options.Pair` each have a criterion; `ReconcileAll` strips only in the walk, as D3 says.
- D1's values match loomyard's hub files, and every fan and lens it names exists in the `lenses` template.
