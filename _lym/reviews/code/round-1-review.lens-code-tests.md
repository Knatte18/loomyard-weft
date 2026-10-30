```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test row that pins it, or none. I made no tool calls and judged from the inherited context.

**Pinned**
- `ReadyFor` for an absent record, the beam's own record and another beam's record: `TestReadyFor` (`internal/pawlengine/ready_test.go`).
- `WriteReady` and `ReadyFor` create the lock directory and round-trip the slug: `TestReadyFor`, indirectly.
- `markReady` refusals for a task, a grouping project and a beam with no pair here, each opened with `lym beam ready:`: `TestMarkReady` rows `TaskRefusedNamingItsOwnLanding`, `GroupingProjectRefusedNamingCreate`, `BeamWithNoPairHereRefusedNamingCreateAndAdopt`.
- `markReady` writes one record with `Slug` and `SetAt` for an unflagged beam, and nothing for a flagged beam, which reports `already`: `TestMarkReady` rows `UnflaggedBeamWritesOneRecordNamingIt`, `FlaggedBeamWritesNothing`.
- `land` refuses an unflagged beam naming `lym beam ready b`: `TestLand` row `UnflaggedBeamRefusedNamingReady`.
- A flagged beam lands as before: the existing `TestLand` rows, since `newLandWorld` flags `b`.
- `landBeam`:
  - lands once per episode with no notice;
  - a reading that does not pass takes no step and posts no notice;
  - a parent owned elsewhere posts a notice naming the owner and `lym adopt`, and lands nothing;
  - a refused landing posts the refusal once;
  - the episode clears and re-arms;
  - the episode survives a restart;
  - the prime's dobby does nothing;
  - a failed post leaves the episode open and the next tick retries;
  - a failed read keeps the episode.

  All of these are rows of `TestLandBeam` (`internal/dobbyengine/land_test.go`).
- `pawlRecheckLines` adds the unflagged line using the beam's slug: `TestPawlRecheckLines` row `AnUnflaggedBeamNamesReady`.
- A flagged beam with nothing else wrong yields no lines: the `NothingBlocksAndBothOwnersAreHere` row.

**Findings**

1. **MEDIUM, design.** `internal/beamcli/dobbydeps.go` (`beamLandReading`, diff hunk near line 186): no test pins that an unflagged beam reads as not passing before any other read. That is the one seam that stops dobby from landing a new empty beam, which is the bug this task fixes. The `TestLandBeam` rows use a fake `CanLand`, so breaking `beamLandReading`'s ready check fails no test. The same goes for the `Ready` and `MarkReady` wiring in `internal/beamcli/deps.go` (the path, the lock path, the `fabricengine.CommitAnchoredPaths` commit and the commit message). Evidence: no `beamLandReading` or `hubLandDeps(...).Ready` test appears in the diff or the context.

2. **MEDIUM, design.** `internal/beamcli/dobbydeps.go:172-175` (`deps.Land`): no test shows that dobby's `Land` runs the `land` sequence and so writes the same land request as `lym beam land`, nor that a refusal from it (owner, run, flag, work remaining) is returned and not swallowed. Only the fake `Land` seam is exercised in `TestLandBeam`. Evidence: the `land_test.go` diff has no test through `deps.Land`.

3. **LOW, scope.** `internal/beamcli/land.go` (`land`): the ordering the focus names, with the unflagged refusal after the owner checks and before the `Refusal` seam, is not pinned. `UnflaggedBeamRefusedNamingReady` leaves `w.refusal` nil and the owners fine, so moving the ready check anywhere still passes. Fix with a row that sets both an owner refusal and an unflagged beam (expecting the owner refusal), and a row that sets `w.refusal` and an unflagged beam (expecting the ready refusal, with `refusedFor` empty).

4. **LOW, scope.** `land` and `markReady` have untested error paths: `deps.Ready` returning an error (both verbs) and `deps.MarkReady` returning an error (`markReady`). The way-forward text of those refusals has no test. `PATTERN-refusal-way-forward` asks that each new refusal's way forward be reached by a test. Evidence: `landWorld.deps()` in the diff always returns a nil error from `Ready` and `MarkReady`.

5. **LOW, scope.** `internal/loomcli/pawlwiring.go` (`pawlRecheck`): the new read of `pawlengine.ReadyFor` at `loomengine.BeamReadyPath` is untested, since `TestPawlRecheckLines` covers only the pure `pawlRecheckLines`. The position of the unflagged line (last, after the parent-owner line) is not pinned, because the `EveryFailureGivesItsOwnLineInOrder` row, as the diff shows it, does not set `unflagged`.

6. **LOW, scope.** `internal/loomengine/config.go` (`BeamReadyPath`, `BeamReadyLockPath`): no test in the diff pins that the record sits under `LandingDir` and the lock at the mirrored `.lym/landing/` path. This is `PATTERN-durable-vs-ephemeral-state`'s stated outcome. Evidence: the `TestReadyFor` paths are hand-built in the test, not taken from `loomengine`.

7. **NIT, scope.** `markReady`'s result keys `record` (the path) and `way_forward` for the first-write case are not asserted; the `TestMarkReady` assertion covers only `already` and the `lym batten status b` substring. The `ReadyFor` error path for a corrupt record is also untested.

8. **NIT, scope.** `internal/dobbyengine/land.go` (`recordLand`): a failed state write after a successful `Land` is only warned about and has no test. The next tick would then call `Land` again, and a second land request would be written.

**Not flagged:** there is no duplicate test; the rewritten `TestLandBeam` replaces the old notice tests, and each row covers one behaviour.

**Tool calls:** none made. Everything came from the diff and files already in context.
