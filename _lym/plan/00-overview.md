---
format: 5
approved: false
language: go
---

# Plan: lym machine name shows and sets the machine name and sweeps owners across a rename

Add the module `machine` with `lym machine name`, which shows this machine's name, sets it with a collision check across every swept hub, and rewrites the machine file so the former names are kept.
After a rename it sweeps each hub's owners recorded under a former name to the new one, through a replace set the owner write gains.
The owner write's change comes first, then the hostname export the parent's added decision records, then the verb, then its sweep.

## Card Index

1 — owner-replace-set — the owner write gains a replace set selecting former-name owners after its pull
2 — machinename-hostname-name — machinename exports the hostname rule's name
3 — machine-name-verb — lym machine name shows and sets the name with its collision check and file write
4 — machine-name-sweep — lym machine name sweeps owners on every swept hub and names the recoveries

## Shared Decisions

- Tests reach the machine file and the hubs only through `internal/machinecli`'s `Deps` (D8): tier0 rows fake the hubs in memory, and one tier1 test drives real hubs.
- The board directory a hub's owner calls take is the one the existing owner callers in `internal/hubgeom` pass.

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`.
