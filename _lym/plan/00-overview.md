---
format: 5
approved: false
language: go
---

# Plan: A machine registry and adopt holds in Bolt, and a catch-up from origin after every absence

`board.json` gains a `machines` registry and a `holds` list, each written only by its `boardengine` setter with a synchronous push.
A random machine id in `machine.json` gives each machine one identity, and every comparison of a recorded machine name with this machine goes through the registry, so a former name reads as its machine (D3).
After an absence (a sleep the dobby sees, a dobby start, a hand pass) a catch-up from origin runs before any act rule answer: it fetches Bolt and the local pairs, registers, pushes owed commits and hands over runs another machine owns, and the origin-owed marker retires (D4).
An adopt hold in Bolt holds the whole hub through the act rule and machine-local `bolt-hold` entries, the `owner` pause caller retires with every writer, and the upgrade's first read turns standing adopt entries into pause intents and one Bolt hold (D5).

## Card Index

1 — fixtries-package — move the fix-tries reader and writer into internal/fixtries
2 — machine-identity — machine.json gains an id and forgotten names; ReadIdentity reads them
3 — bolt-registry-holds-records — board.json's machines and holds records, their readers and the registry comparison
4 — machine-registration-setter — the registry setter with every D3 refusal, and the machine id write
5 — bolt-hold-write — the one hold write, synchronous
6 — owner-write-registers — every owner write registers this machine in its own commit
7 — owned-elsewhere-through-registry — OwnedElsewhere takes the registry; every name comparison moves onto it
8 — dobby-start-owner-flags — OwnedWorktrees, EnsureLive and the dobby's owner read follow registry flags
9 — this-machine-names-registry — the pair push's and landings' name set comes from the registry
10 — machine-name-registers — lym machine name records the rename in the registry and keeps or replaces the id
11 — forget-former — lym machine name --forget-former drops a former name and lists what names it
12 — bolt-hold-caller — the bolt-hold pause caller and its wording in every pause reader
13 — act-rule-hold — ReadRun fills Held from Bolt; the held refusal names the hold's machine
14 — act-verbs-pull-bolt — loom resume and tender resume fetch Bolt before the act rule
15 — tender-bolt-hold — write and drop bolt-hold entries; tender resume's hold rules
16 — held-tasks-not-started — no queued task is claimed under a hold
17 — hand-over-owned-elsewhere — pairteardown hands over a run another machine owns
18 — catch-up-engine — dobbyengine.CatchUp over seams
19 — pause-intent-entries — pause entries from intents, and a pause intent whose push must land
20 — hand-pass-catch-up — a run's own pass catches up; the origin-owed marker retires
21 — dobby-catch-up — the dobby catches up after every absence and syncs bolt-hold entries
22 — upgrade-first-read — standing adopt entries become pause intents and one Bolt hold
23 — retire-owner-pause-callers — no code path writes the owner caller
24 — delete-tender-owner-pause — delete PauseOwnedElsewhere and its process end
25 — pattern-owner-machine — PATTERN-owner-machine's changed line and background

## Shared Decisions

### Decision: identity-beside-the-name

- **Decision:** `machinename.Identity` (id, name, former, forgotten names) is the "here" every comparison takes; `boardengine.Registry.ReadsAsHere` is the one rule and `boardengine.OwnedElsewhere(owner, registry, here)` its owner form.
  `machinename` stays a leaf that writes no file; the id is written by `boardengine.EnsureIdentity` under the machine file's lock and by `lym machine name`.
- **Rationale:** the leaf entry admits only `standalonestate`, so the lock-taking writer lives in `boardengine`, which owns the registry the id is registered in.
- **Applies to:** cards 2–11

### Decision: the engine never compares names

- **Decision:** `dobbyengine` receives owned-elsewhere flags and the catch-up's and upgrade's seams; the hub wiring (`hubgeom`, `beamcli`, `battencli`) judges through the registry and calls fabric and teardown.
- **Rationale:** the Told-Geometry allowlist keeps fabric, teardown and CLI packages out of `dobbyengine`.
- **Applies to:** cards 8, 18, 20–22

### Decision: absences and their observers

- **Decision:** the dobby loop is the long-lived process that observes a sleep and runs the catch-up for its worktree's runs; every pass that is not a continue pass runs the catch-up of its own run, which covers the hand pass.
  The orch watcher (`internal/orchengine`, outside this part's packages) gets no sleep watch: the dobby of the same worktree observes the same sleep and its catch-up covers the worktree's runs.
- **Rationale:** batten cannot tell a hand pass from a dobby's full pass, and a full pass already runs the run-scope reconcile, so its own run's fetch costs little; one observer per worktree avoids two catch-ups racing over one pair.
- **Applies to:** cards 20, 21

### Decision: caches dropped by the catch-up

- **Decision:** the "caches of the runs it touched" are the dobby's per-run memory (its state record's owed full pass and its in-memory child, lift and cover readings); the catch-up owes each touched run a full pass and forgets that memory.
- **Rationale:** `ReadRun` reads no cache (`TestReadRun_ReadsTheSameStateWithoutTheCache`), so the dobby's memory is what a catch-up can leave stale.
- **Applies to:** card 21

### Decision: adopt's own pause step stays

- **Decision:** `adoptcli`'s pause step (`pauseEntries`, which writes `AdoptCaller`) is left to the adopt part (D6), as D5 says; this part keeps `AdoptCaller`, `HoldCaller` and `OwnerCaller` declared for the upgrade's first read, and adoptcli changes only its owner comparisons and its fix-tries calls.
- **Rationale:** D5: "The adopt and legacy adopt-hold callers' writers retire with adopt's pause step (D6)".
- **Applies to:** cards 1, 7, 22, 23

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, a Finalize past its merge included, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf` and Bolt's machine registry; after an absence, a catch-up from origin first.
