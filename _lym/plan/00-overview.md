---
format: 5
approved: false
language: go
---

# Plan: lym machine name shows and sets the machine name and sweeps owners across a rename

Add the module `machine` with `lym machine name`, which shows this machine's name and sets it in the machine file, refusing a name another machine owns entries under on any swept board.
After a rename it sweeps every registered hub, and the hub of the working directory, so owners recorded under this machine's former names follow it.
The owner write gains a replace set that selects those owners after its own pull, and `internal/machinename` exports the hostname rule's name, per the parent's added-after-Discussion decision.

## Card Index

1 — machinename-hostname-name — export the hostname rule's name from internal/machinename
2 — owner-replace-set — the owner write's replace set rewrites former-name owners after its pull
3 — machine-name-verb — lym machine name shows the name, checks collisions and writes the machine file
4 — owner-sweep — lym machine name sweeps each hub's owners and reports failures and recoveries

## Shared Decisions

### Decision: seams

- **Decision:** `internal/machinecli` takes the machine file's path, the hostname rule's name and the registered hubs through `nameDeps`, and the working directory through the command context `RunCLIIn` seeds.
  Production fills them from `machinename.FilePath`, `machinename.HostnameName` and `hubregistry.Live` over `standalonestate.UserStateDir()`.
- **Rationale:** D8: no test writes the developer's machine file or a real hub's Bolt, and no test environment variable changes.
- **Applies to:** cards 3 and 4

### Decision: the hostname is read only in internal/machinename

- **Decision:** `lym machine name` takes the hostname rule's name from `machinename.HostnameName` (card 1), never `os.Hostname`.
- **Rationale:** the decision record's added-after-Discussion entry; `PATTERN-owner-machine`'s sole-source scan.
- **Applies to:** cards 1 and 3

### Decision: empty names match nothing

- **Decision:** an empty `names` entry is dropped by the file rewrite, kept out of the replace set, and matched by no owner selection in the owner write.
- **Rationale:** an empty recorded owner means no owner; matching it would take every unowned beam and an unclaimed prime.
- **Applies to:** cards 2, 3 and 4

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`.
