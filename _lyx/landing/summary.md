# lyx gate test --upto N with manual tiers, and lyx gate status

Running a hub's tiers up to a level meant naming each one, as the push gate's `lyx gate test --tier 0 --tier 1 --tier 2` did.
`lyx gate test` was also the gate module's only verb, so a run waiting for a slot gave no reason.

## What changes

- `lyx gate test --upto N` runs every declared tier numbered 0 through N in ascending order, and drops a tier another selected tier `covers`, as `--tier` does.
- A tier may carry `manual: true` in `gate.yaml`.
  `--upto` skips a manual tier and names it on stderr; the tier runs only when `--tier` names it.
- `lyx gate status` is a new read-only verb.
  It reports whether the directory is in a hub, the pool's slot count, whether a multi-slot waiter holds the drain, and one entry per held slot with its slot number, worktree, site, tier, pid and start time.
  It takes no slot and takes no `-C` flag; it resolves the hub from the working directory.
- `CLAUDE.md`'s push gate is now `lyx gate test --upto 2`, and `docs/overview.md`, `internal/gatecli/doc.go`, `internal/gateslot/doc.go` and the `gate.yaml` template describe the new flag, key and verb.

There is no separate "every tier" flag: `--upto` with a number above the highest declared tier serves that need, manual tiers left out.

## Design decisions

- `--upto` unions with `--tier`: the run list is the `--tier` values plus every declared non-manual tier from 0 through N, and the covered-tier drop runs over that union.
  `--upto 2 --tier 3` is therefore the way to add a manual tier to a range.
  `--upto` selects only declared tiers, so a gap in the numbering plans only the declared ones.
- `--upto` alone replaces the tier0 default; with neither flag the default is still tier0.
- A negative `--upto` is refused before hub resolution, with a way forward.
- `manual: true` on `tier0` is refused at load, naming the key: tier0 is the default tier of a call with no tier flag, so it cannot be hand-run only.
- A gate list (`gates.<name>`) may name a manual tier, since a list names its tiers explicitly, as `--tier` does.
- `gateslot.Holder` gains `Slot`, which `Holders` fills from the record's file name and never writes into the record, so records already on disk stay valid and the record's YAML gains no key.
- `gateslot.Pool` gains `Draining`, which reports whether the drain lock is held without creating the lock file.
- Outside every hub, `status` answers ok with `in_hub: false` and no slots, since the tiers run unslotted there.

## Known gap

A holder record read between a slot's lock and its record write is skipped, as `Holders` already does, so `status` can miss a slot taken in that instant.

## To do by hand

- Set `manual: true` on the loomyard hub's `tier3` after landing, a hub config write outside this change.
  Until then `--upto` with a number of 3 or more would include it.
- Run `lyx gate status` in a hub with a held slot to see the output against a live pool.
