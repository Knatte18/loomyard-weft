---
format: 5
approved: false
language: go
---

# Plan: lym machine name shows and sets the machine name and sweeps owners across a rename

This part adds the module `machine` with the verb `lym machine name`.
With no argument the verb shows this machine's name, its source, the machine file's path and the former names.
With a name it validates the name, refuses one a swept board records for another machine, rewrites the machine file keeping every former name,
and rewrites, on every swept hub, each owner recorded under a former name to the new name.
The owner write gains a replace set for that sweep, selected after the write's own pull, so no `Takeover` is needed.

## Card Index

1 — owner-replace-set — the owner write takes a replace set and rewrites the prime and open beams recorded under it
2 — hostname-name — `internal/machinename` exports the hostname rule's name of this machine
3 — machine-module-show — the module `machine`, mounted, with `lym machine name` showing the name
4 — owner-sweep — the hubs to sweep, the one owner write per hub, its outcomes and the recovery lines
5 — set-name — `lym machine name <name>`: validation, the collision check, the file write and the sweep

## Shared Decisions

- **Departure from the record's Scope: card 2 changes `internal/machinename`.**
  D5 has the verb take the hostname rule's name through `machinename.FromHostname` even when a machine file exists.
  `PATTERN-owner-machine`'s enforcement, the sole-source scan in `internal/machinename`, lets no production file outside that package read the hostname,
  and the package exports no reader of it.
  The constraint wins over the Scope list: `internal/machinename` gains `HostnameName`, and `internal/machinecli` never reads the hostname, as a call or as a function value.
  The parent was asked to record this and answered that the record stays frozen and the plan states the departure here.
- **Assumption: the `PATTERN-cli-cobra` background gains a `machinecli` bullet with no declared PATTERN change.**
  Its Package naming list names every `<module>cli` without an engine package, and `internal/machinecli` is one.
  The record declares no `PATTERN-cli-cobra` change and the entry's overview line stays as it is, so card 3 adds the bullet as the docs of the package it creates.
- **`PATTERN-owner-machine`'s declared change is to its background.**
  The entry's overview line keeps its text; card 1 fences it and rewrites the background clause on what the owner setter takes from another machine.
- **`internal/machinecli` has no engine package.**
  It calls `machinename`, `boardengine`, `fabricengine`, `hubregistry` and `lymcwd` through the unexported `deps` struct of D8.
  Tier0 tests hand in fakes for every seam; one tier1 test runs the production board seams over `hubforge` hubs under its temporary directory.
- **The sweep lands before the set path.**
  Card 4 adds the sweep as functions no command calls, and card 5 wires the file write and the sweep together,
  so no commit holds a verb that renames the machine without sweeping its owners.
- **The collision check reads every recorded owner**, the prime's and each entry's, done entries included, while the sweep rewrites the prime and open beam projects only.
- **A pull that returns an error or any skip** makes its hub checked locally only; the record names the failed fetch, and a dirty or diverged Bolt is read the same way.
- **Output form.** The verb prints the standard envelope through `internal/output`; a bad file in the show form, and a sweep with a failed or lost hub, print an error envelope carrying the same fields and exit non-zero.

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`.
