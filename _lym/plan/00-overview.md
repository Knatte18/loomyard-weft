---
format: 5
approved: false
language: go
---

# Plan: lym machine name shows and sets the machine name and sweeps owners across a rename

Add the module `machine` with `lym machine name`.
With no argument it shows this machine's name, its source, the machine file's path and the former names.
With a name it validates it and refuses a name another machine's owners hold on any swept hub.
It then rewrites the machine file keeping every name, and sweeps each hub's owners from the former names to the new one.
The owner write gains a replace set that selects those owners after its own pull, so an owner only origin held is swept too.

## Card Index

1 — owner-replace-set — give the owner write a replace set that rewrites former-name owners selected after its pull
2 — hostname-name — export the hostname rule's name from internal/machinename
3 — machine-module-show — add the machine module and lym machine name's show form, wired under the root
4 — set-name — validate a new name, check it against every swept board's owners, and rewrite the machine file
5 — owner-sweep — sweep every hub's former-name owners to the new name and report each hub and the recoveries

## Shared Decisions

### Decision: replace-set-field

- **Decision:** `boardengine.OwnerRequest`'s replace set is the field `Replace []string`.
  The owner write ignores its empty entries and selects open beam projects and the prime only.
- **Rationale:** an empty entry would match every unrecorded owner,
  and D6 keeps a done entry's recorded owner.
- **Applies to:** cards 1 and 5

### Decision: hostname-reader-outside-scope

- **Decision:** card 2 adds `machinename.HostnameName` in `internal/machinename`, a package outside the decision record's Scope list.
  This is a plain departure from that list.
- **Rationale:** D5 has the verb take the hostname rule's name as one of this machine's names, over a machine file too.
  No function the record's Takes line names returns it while a file exists.
  `PATTERN-owner-machine` keeps `internal/machinename` the only production reader of the hostname,
  so `internal/machinecli` cannot read `os.Hostname` itself.
  The parent declined to record an addition and directed the departure.
- **Applies to:** cards 2, 3 and 4

### Decision: seams-for-tests

- **Decision:** `internal/machinecli`'s unexported `nameDeps` carries the file path, the hostname rule's name, the registered hubs, the working directory and a before-push hook.
  `commandWith` takes them, so every test hands in paths and hubs under its temporary directory (D8).
- **Rationale:** no test may write the developer's machine file or a real hub's Bolt.
- **Applies to:** cards 3, 4 and 5

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`.
