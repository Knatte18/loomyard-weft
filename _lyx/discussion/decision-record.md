# Decision record — gate-tier-verbs

## Goal

Let a caller run a hub's tiers up to a level with one flag, keep hand-run tiers out of that flag, and show who holds the hub's gate slots.

## Scope

- `lyx gate test --upto N`: every declared tier numbered 0 through N, ascending, covered tiers dropped as `--tier` does.
- A tier key `manual: true` in `gate.yaml`, which `--upto` skips.
- `lyx gate status`: a read-only verb listing the held slots of the hub's gate-slot pool, by run and tier, with each start time.
- The docs the change touches: `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, `docs/overview.md`'s module line and `CLAUDE.md`'s push-gate call.
- Out of scope: setting `manual: true` on the loomyard hub's `tier3`, a hub config write the orch makes after landing.

## Decisions

- **`--upto` unions with `--tier`.**
  The run list is the `--tier` values plus every declared, non-manual tier from 0 through N, then the covered-tier drop runs over that union.
  Rationale: `--upto 2 --tier 3` is the one way to add a manual tier to a range, and the union needs no new refusal.
  Bound: it selects only tiers the hub declares, and a manual tier joins only when `--tier` names it.
- **`--upto` takes any N of 0 or more; a negative N is refused before hub resolution with a way forward.**
  Rationale: a number above the highest declared tier means every non-manual tier, which is the "every tier" use the board entry names.
- **A manual tier `--upto` skips is named on stderr**, as a covered tier is.
  Rationale: the caller sees why a declared tier did not run.
- **`manual: true` on `tier0` is refused at load, naming the key.**
  Rationale: tier0 is the default tier of a call with no tier flag, and every hub needs it, so it cannot be hand-run only.
- **A gate list (`gates.<name>`) may name a manual tier.**
  Rationale: a gate list names its tiers explicitly, as `--tier` does.
- **`lyx gate status` answers an ok envelope: whether the directory is in a hub, the pool's slot count, whether a multi-slot waiter holds the drain, and one entry per held slot with its slot number, worktree, site, tier, pid and start time.**
  Outside every hub it answers ok with `in_hub: false` and no slots, since the tiers run unslotted there.
  Rationale: the drain explains a wait while some slots are free, and the slot number lets the reader match the lock files.
- **`gateslot.Holder` gains a slot number that `Holders` fills from the record's file name, never written into the record.**
  Rationale: records already on disk carry no number, and the file name is its single source.
- **`gateslot.Pool` gains `Draining`, reporting whether the drain lock is held.**
- **`CLAUDE.md`'s push gate becomes `lyx gate test --upto 2`.**
  Rationale: the same tiers, named the new way; the loomyard hub's `tier3` sits above 2, so it stays out without the manual mark.

## Constraints

- `PATTERN-test-tiers`: tests run only through `lyx gate test`.
- `PATTERN-gate-slot`: status only reads the pool; it takes no slot.
- `PATTERN-refusal-way-forward`: every new refusal names its way forward.
- `PATTERN-tolerant-records`: the holder record's YAML gains no key.
- `PATTERN-comment-line-breaks`: one sentence per comment line.

## Auto-mode assumptions

- `status` takes no `-C` flag: it resolves the hub from the working directory, since the pool is hub-wide.
- `status` carries the operator audience annotation, as `test` does.

## Open risks

- A holder record read between a slot's lock and its record write is skipped, as `Holders` does today; status can miss a slot taken in that instant.

## Acceptance criteria

- `--upto N` plans every declared non-manual tier from 0 through N, ascending, with covered tiers dropped and named.
- `--upto` skips a manual tier and names it on stderr; `--tier` with that number runs it, alone or beside `--upto`.
- `--upto` with a gap in the declared numbers plans only the declared ones.
- A negative `--upto` is refused before hub resolution with a way forward.
- `manual: true` loads on a tier above 0 and is refused on tier0.
- `lyx gate status` in a hub lists each held slot's number, worktree, site, tier and start time, the slot count and the drain state.
- `lyx gate status` outside every hub answers ok with `in_hub: false`.

## Notes for the plan writer

- Tier selection lives in `planTierRuns` and `listedTiers` in `internal/gatecli/tiers.go`.
- `Pool.Holders` in `internal/gateslot/pool.go` already filters stale records.
