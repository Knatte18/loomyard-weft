---
format: 5
approved: true
language: go
---

# Plan: lym machine name shows and sets the machine name and sweeps owners across a rename

`lym machine name` shows this machine's name, or sets it in the machine file and rewrites, on every swept hub, the owners recorded under this machine's former names.
The owner write gains a replace set that selects those owners after its own pull, so the sweep reaches owners only origin held.
Tests hand in the machine file's path and the hubs through seams, so none touches the developer's file or a real hub.

## Card Index

1 — owner-replace-set — `boardengine.OwnerRequest` gains the replace set, selected after the write's pull, and the owner pattern names it
2 — hostname-name — `machinename.FromThisHost` exports the hostname rule's name of this host
3 — machine-name-show — new module `machine`: `lym machine name` shows the name, wired under the root
4 — machine-name-set — `lym machine name <name>` validates, checks collisions, writes the file and sweeps every hub

## Shared Decisions

- The verb needs the hostname rule's name over any machine file (D5), and the decision record names `machinename.FromHostname`, which takes a hostname string.
  `PATTERN-owner-machine`'s background makes `internal/machinename` the only production reader of the hostname, enforced by its sole-source scan, so `internal/machinecli` cannot read the hostname itself.
  The constraint wins: card 2 adds `machinename.FromThisHost`, a package the record's scope list does not name, rather than reading the hostname in `internal/machinecli`.
- `internal/machinecli` finds the hub of the working directory itself (card 4's `hubOfDir`) rather than through `preflight.ResolveMode` or `internal/cliwire`, which `PATTERN-cli-cobra`'s sole-wiring rule names for standalone-capable CLIs.
  `lym machine name` is no standalone-capable module: it resolves no `--target-dir`, repository root or mode-derived state, and needs only an optional hub path.
  D5 has it run from any directory, and `ResolveMode` refuses a wired hub worktree's subdirectory, so the verb cannot route through it.
- `internal/machinecli` takes every seam through a `deps` value that `newCommand` closes over, rather than a package-level seam variable, so its tests run in parallel with their own file paths and hubs (D8).

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`.
