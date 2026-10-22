# lyx gate test --upto N with manual tiers, and a lyx gate status verb

Running a hub's tiers up to a level used to mean naming each one, as in `lyx gate test --tier 0 --tier 1 --tier 2`.
Nothing showed which gate slots were taken, so a run waiting for a slot gave no reason.

## What changes

- **`lyx gate test --upto N`** adds every declared tier numbered 0 through N to the `--tier` values.
  It skips gaps in the declared numbers, and covered tiers are dropped as before.
- **`manual: true` on a tier in `gate.yaml`** keeps that tier out of an `--upto` range.
  `--upto` names each manual tier it skips on stderr; the tier runs only when `--tier` names it, alone or beside `--upto`.
  `manual: true` on `tier0` is refused at load, since tier0 is the tier a call with no tier flag runs.
- **A negative `--upto`** is refused before hub resolution, with a way forward.
- **`lyx gate status`** is a read-only verb over the hub's gate-slot pool.
  It answers an ok envelope with `in_hub`, the pool's `slots` count and `draining`, which says whether a multi-slot acquire holds the drain lock and so makes new single-slot runs wait while a slot is free.
  It also lists `held`: one entry per held slot, oldest first, with its `slot` number, `worktree`, `site`, `tier`, `pid` and `started` time.
  Outside every hub it answers `in_hub: false` with nothing held, since tiers run unslotted there.
- **`gateslot`** adds the two reads `status` needs.
  `Holder.Slot` is filled by `Pool.Holders` from the record's file name and is never written into the record.
  `Pool.Draining` reports the drain lock, and creates no file in a pool that never had one.
- **Docs:** `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, the tier-key comment in `internal/gateslot/template.yaml`, and the `docs/overview.md` module line.
  The push gate in `CLAUDE.md` now reads `lyx gate test --upto 2`, which runs the same tiers as before.

## Design decisions

- `--upto` and `--tier` combine as a union, so `--upto 2 --tier 3` adds a manual tier to a range in one call.
- `--upto` accepts any N of 0 or more; an N above the highest declared tier means every non-manual tier, which covers the "every tier" need without a separate flag.
- A gate list (`gates.<name>`) may still name a manual tier, since it names its tiers explicitly, as `--tier` does.
- `status` takes no `-C` flag, since the pool is hub-wide and resolves from the working directory.
  It does not list waiters, because wait records are kept per worktree.

## Out of scope

Setting `manual: true` on the loomyard hub's `tier3` is a hub config write the orch makes after landing.

## Tests

- `TestPlanTierRuns` gains `--upto` rows: a range with a gap and a skipped manual tier, `--upto 0`, `--tier` adding a manual tier beside `--upto`, and a covered tier dropped inside a range.
- `TestGateTest_RefusesBeforeResolvingTheHub` gains the negative `--upto` refusal.
- `TestLoadConfig` gains a manual tier that loads and the refused manual `tier0`.
- `TestHolders_NamesLiveHoldersAndDropsRecordsOfFreeLocks` asserts the slot number, and `TestDraining_ReportsTheDrainLockWithoutCreatingIt` covers the drain read.
- `TestGateTest_Scenario` runs `lyx gate status` in a hub with two held slots, one held by a tier run, and outside every hub.
