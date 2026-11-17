# Card 1 — Bolt replay carries holds and the registry

**Create:**
- `plan:internal/boardengine#parseReplayHolds` -> `func parseReplayHolds(data []byte) ([]Hold, Registry, error)`

**Edit:**
- `internal/boardengine#FabricReplay.Apply`
- `internal/boardengine#replayRun`
- `internal/boardengine/doc.go`
- `internal/boardengine#TestFabricReplay`
- `internal/boardengine#replayRow`

**Uses:**
- `internal/boardengine#Registry.sameMachine`
- `internal/boardengine#Hold`
- `internal/boardengine#boardJSONWithDraft`

**Intent:**
A rejected Bolt push replays the local commit onto origin's tip through `FabricReplay.Apply`, which today re-applies entries, the prime owner and design files only.
The `holds` list and the `machines` registry of the commit are dropped silently, so a replayed adopt write would land its owners with no hold and no registration.
D6 step 1 needs the replay to carry both, and to refuse an adopt commit whose hold meets another machine's hold that origin gained.

`parseReplayHolds` reads `holds` and `machines` from one side's `board.json` bytes, nil data holding none.
`Apply` parses them for the merge base, the commit's parent and the commit, and judges the holds before any entry change:

- Holds: when the commit's holds differ from its parent's, the commit made hold changes.
  When origin's store holds a hold of another machine (compared through `Registry.sameMachine` over origin's registry) that the commit's parent did not hold, the commit's whole `board.json` change is refused:
  no entry, prime-owner, hold or registry change of that commit is applied, and one `Refused` finding with entry `holds` names that hold's machine and time.
  The hold check runs before the entry changes, so an adopt commit lands all or nothing.
  Otherwise each hold of the parent that the commit removed or replaced is matched in origin's list by deep equality with the parent's hold:
  a removed hold is removed from origin's list, and a replaced hold (the same list position in parent and commit, its content changed, as a take-hold rewrite or `FixHold` makes) is replaced by the commit's.
  A parent hold origin no longer holds is reported `Lost` and its change is not applied.
  Each hold the commit added is appended.
- Registry: each machine entry the commit added or changed is set on origin's registry, unless origin changed the same id's entry since the merge base, which keeps origin's and reports a `Lost` finding.
- Any applied hold or registry change sets `applied`, so the store is saved.

`replayRun` gains the parsed holds and registry of the three sides.
A commit that changes neither key replays exactly as today, even when origin gained a hold.
doc.go's replay paragraph names the holds and the registry among what the replay carries, and the whole-commit refusal.

`replayRow` gains `holds` and `machines` per side, nil leaving the key absent, and the rows render a side's `board.json` with them beside the tasks and prime owner (`boardJSONWithDraft`'s shape plus the two keys).
`TestFabricReplay` (tier 0) gains rows:
`AHoldTheCommitAddedLandsOnOrigin`, `AHoldRemovedByTheCommitLeavesOrigin`,
`AHoldTheCommitRewroteToThisMachineReplacesOriginsCopy` (a take-hold rewrite whose machine changes),
`AnAdoptCommitMeetingAnotherMachinesNewHoldIsRefusedWhole` (the commit also changes an entry's owner and the prime owner that origin left alone; asserts origin's owners, prime owner, holds and registry unchanged and one refused finding naming the machine),
`ACommitTouchingNoHoldReplaysOverAHoldOriginGained` (the commit changes an entry only; it lands and origin's hold stays),
`AHoldOfTheSameMachineUnderAFormerNameIsNotAnotherMachines`,
`ARegistryEntryTheCommitAddedLands`, and `ARegistryEntryOriginChangedSinceTheBaseIsLost`.

**ImpactSummary:** replay.go and its test only; every Bolt write's replay path gains hold and registry carry, and a commit touching neither replays unchanged.

**Commit:** 1: Replay a Bolt commit's holds and registry, refusing an adopt commit over another machine's new hold
