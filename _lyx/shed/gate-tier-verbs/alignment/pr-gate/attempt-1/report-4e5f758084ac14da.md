---
verdict: APPROVED
---

## Alignment at pr-gate: gate-tier-verbs, PR 815 at a3b683ab4

The measure is board entry `gate-tier-verbs`.
The decision record has no `Added after Discussion` section, so the board body stands unamended.

### Board points against the diff

- `lyx gate test --upto N` runs every declared tier numbered 0 through N, ascending, with covered tiers dropped as `--tier` does: `listedTiers` in `internal/gatecli/tiers.go` adds the declared tiers up to N, sorts them, removes repeats and passes them through the existing covered-tier drop in `planTierRuns`; the flag is registered in `internal/gatecli/cli.go`.
- A tier may carry `manual: true` in `gate.yaml`, `--upto` skips it, and only `--tier` runs it: `Tier.Manual` and `tierFile.Manual` in `internal/gateslot/tiers.go`; `listedTiers` leaves a manual tier out of the range and names it on stderr unless `--tier` names it.
- No separate "every tier" flag: none is added, and `--upto` accepts any N of 0 or more.
- `lyx gate status` is a read-only verb showing which slots are taken, by which run and tier, and since when: `internal/gatecli/status.go` reads `Pool.Limits`, `Pool.Holders` and the new `Pool.Draining`, takes no slot, and reports each holder's slot, worktree, site, tier, pid and start time.
- Out of scope, setting `manual: true` on the loomyard hub's `tier3`: not done; only the shipped `template.yaml` comment names the new key.

The additions beyond the bare points stay inside the entry's direction:
refusing `manual: true` on `tier0`, refusing a negative `--upto`, the slot number and drain state in the status envelope, and the doc updates in `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, `docs/overview.md` and `CLAUDE.md`'s push-gate line, which the project's same-commit docs rule requires.
No change outside the entry's scope was found.

### Open risks carried over

The record carries one open risk: `lyx gate status` can miss a slot taken between its lock and its holder record write.
It names no parent correction, so it does not escalate.

### Overlap with peers and parent commits

`lyx loom peers` lists no parent commits and three peers: `machine-handover` (row 5), `quarry-csharp` (row 6) and `darn-verify-fixes` (row 7), all with no changed paths yet; this run is at row 21.

- `darn-verify-fixes` changes `internal/impactset`'s rule for the computed scope that `lyx gate test` consumes, and adds a shell-guard test case naming `lyx gate test --tier 2`.
  It does not touch tier selection, `gate.yaml` tier keys or the slot pool, so there is no overlap in behaviour.
- `quarry-csharp` only names `lyx gate test` as the route for its tests and as a verify command whose package arguments it leaves unchecked: no overlap.
- `machine-handover` does not mention the gate module: no overlap.

No overlap was found, so no landing order applies.
If one shows up later, this run lands first, being furthest along at row 21 against rows 5 to 7.

### Superseded entries

None.
Board entry `gate-test-repeats` also concerns the gate tiers, but it is about repeated suite runs and disjoint tiers, which this run does not address.
