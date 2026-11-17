```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. I checked what the diff's changes force outside the lines it touches.

## Per change: set found, covered or not

1. **Removed dobby owner pause** (`PauseOwned` and `LiftOwnerPause` seams, `OwnerPauseFailed`, `recheckOwnerPause`, `ownerPauses`, `tendercli.PauseOwnedElsewhere`, `orchcli.pauseOwned`).
   - A grep over `.go`, `.md`, `.yaml` and `.tmpl` outside `_lym`, `.lym` and `docs/research` finds no remaining caller. Covered.
   - The `orchcli` test harness fields `pauseOwned` and `pauseOwnedErr` and `pauseOwnedRuns` keep their names. The decision record's card 23 entry says the test-local fields keep their names. Covered.
2. **Retired origin-owed marker** (`OriginOwed`, `passFullNoReopen`, the `resumereconcile.go` marker).
   - No production or test reference remains.
   - The matches under `docs/research/` are dated research notes, so I did not treat them as stale. Covered.
3. **`OwnedElsewhere(owner, registry, here)` and the `Owner`, `PrimeOwner` and `BeamPair.Elsewhere` seams.** Every caller in the diff was updated: boardengine, hubgeom, dobbyengine, beamcli, battencli, orchcli, pairteardown, tendercli and adoptcli. No raw name equality showed up in the diffs I read. Covered.
4. **`PATTERN-owner-machine`.**
   - The line in `PATTERN.md:24` and the header line in `pattern/PATTERN-owner-machine.md` carry the declared text exactly. Covered.
   - The `PATTERN-cli-cobra` machinecli import list gained `hubgeom` and `pairteardown`. Covered.
   - The `PATTERN-dobby-runs` Adopt pause rule no longer lists the dobby's owner lift. Covered.
5. **`OwnerCaller` retired with every writer.** Two findings, below.
6. **Commit messages.** The log shows 25 card commits and no `Pattern-Departure:` trailer, so none needs judging. Covered.

## Findings

**F1 — LOW, class `consistency`.** `internal/pauserecord/doc.go:17-31` still describes the retired `owner` caller as live behaviour.
- It says `OwnerCaller` is "the fixed `Caller` of every pause entry that holds a run owned by another machine". Line 19 adds "No code path writes it any more", but the surrounding text (`OwnerEntryExempt` and the parent-lock release judging only it) reads as current design.
- `internal/pauserecord/record.go:22` has the same stale description on the const: "Caller of every pause entry that holds a run owned by another machine".
- The doc and the const comment should say the name is kept only so the upgrade's first read recognises and drops legacy entries (`internal/dobbyengine/upgrade.go:145`).

**F2 — MEDIUM, class `scope`.** Production readers of the retired `owner` caller remain.
- The caller has no writer any more, so each of these is now dead, with no entry able to trigger it:
  - `Coverage.OwnerHeld` at `internal/pauserecord/record.go:242-251`.
  - `OwnerEntryExempt` at `internal/pauserecord/hold.go:254` and `:319`, wired in `internal/tendercli/gate.go:94`.
  - The parent-lock release at `internal/battenshed/innerrun_parentlock.go:36`.
  - `pairteardown.OwnerPauseHeld` at `internal/pairteardown/followup.go:53`, used at `followup.go:131`, `teardown.go:228`, `internal/battencli/owner.go:313` and `internal/battencli/cancelcmd.go:224`.
- Decision D5 says the `owner` caller "retires with every writer". The record's Scope names `CoveredError` and `SleepWatch`, and the Takes line does not name these readers, so removing them may be outside the plan's reach.
- If the plan intended them to stay, the stale wording in F1 and the `owner pause` wording in `followup.go:33-34`, `followup.go:148` and `battenshed/innerrun_parentlock.go:37` is a consistency gap.
- This is a retirement-completeness question, so the handler should decide it.

## Tool calls (3 of 6)
1. grep for removed symbols and `PATTERN.md` owner-machine line: my context had only the diffs, not a repo-wide caller search.
2. grep for owner-pause and `OwnerCaller` wording: to find readers and docs the diff left.
3. `sed` of `pauserecord/doc.go` lines 10-34 plus a grep for `OwnerHeld`, `OwnerEntryExempt` and `OwnerPauseHeld` production callers: the doc context and the callers were not in my context.
