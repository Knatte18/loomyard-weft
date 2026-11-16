---
format: 5
approved: false
language: go
---

# Plan: lym machine name shows and sets the machine name and sweeps owners across a rename

Add the module `machine` with `lym machine name`, which shows this machine's name or sets it.
Setting it checks the name against every swept board, writes the machine file keeping the former names, and sweeps every hub's owners from the former names to the new one.
The sweep rides a new replace set on `boardengine`'s owner write, which selects the owners after the write's own pull.
`internal/machinename` exports the hostname rule's name, per the decision record's added entry, so the verb counts it among this machine's names without reading the hostname itself.

## Card Index

1 — owner-replace-set — the owner write rewrites the owners its replace set names, after its own pull
2 — hostname-name — machinename exports the hostname rule's name
3 — machine-name-show — the machine module and `lym machine name` with no argument
4 — machine-name-set — `lym machine name <name>` checks, writes the file and sweeps every hub

## Shared Decisions

### Decision: the verb restates the name rule over its seam's path

- **Decision:** `internal/machinecli` resolves the shown and current name through `machinename.ReadFile` and `machinename.CheckName` at the path its seam hands in, never through `machinename.Name` or `machinename.Names`.
- **Rationale:** D8 has every test hand in the file's path; `Name` and `Names` read the developer's own file, and D5 has the set path read a bad file itself to repair it.
- **Applies to:** cards 3 and 4

### Decision: the replace set is a field beside Slugs and Prime

- **Decision:** `OwnerRequest.ReplaceOwners` selects entries inside the write, after its pull, and leaves `Takeover` unset; the commit subject names the set, since the message is fixed before the pull.
- **Rationale:** D6 has the write select owners after its own pull, so an owner only origin holds is swept.
- **Applies to:** cards 1 and 4

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`.
