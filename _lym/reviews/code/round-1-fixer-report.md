# Fixer-report, code round 1

## Changes

All four findings fixed as proposed, in `contracts/orchrole/orch-role.md`, in one commit (`4c9d9a7d`, covering F1-F4 because all four edit the same section).

- F1: replaced the producer-filtered `lym batten status` test with one that works. The section now says `lym batten status` reports the batten run's own rows for both kinds, and names the entry's `kind` (`lym board get <slug>`) and the inner producer read with `lym loom status` as the tests. The precondition held: the pawl run is wired through `loomcli`, so `lym loom status` reports it. The cancel/create sentence is unchanged.
- F2: the tell-apart read now opens with its trigger, "When the operator reports that a project ran as a task run, or a stop shows it". Run-loop item 4 is unchanged.
- F3: step 2's `--anchor` now reads "a path relative to the code repo root, tracked in the commit the beam forks from".
- F4: the sentence is now "Dobby starts no project entry, so a release never makes a beam; make one with `lym beam create`."

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
