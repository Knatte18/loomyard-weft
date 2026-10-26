```yaml
lens: coverage
outcome: reported
```

I found 13 coverage defects: 0 BLOCKING, 8 MEDIUM and 5 LOW.

1. MEDIUM — D10 `lyx beam land`: the record says the verb runs from anywhere in the hub, but card 22 keeps it on `runVerb`, so a task pair is still refused.
   - Evidence: `internal/beamcli#runVerb` passes the location through `battencli.ResolveBattenPrime`, which refuses any worktree that is neither the prime nor a beam (`internal/battencli/refusal.go:73-77`).
   - Card 22 changes `land` and `landCmd` but not that resolution, and no `TestLand` row lands from a task pair.

2. MEDIUM — D3 refusals of `lyx beam start` (card 11): three refusals have no test row.
   - The "outside a hub, naming the hub" refusal.
   - The unknown slug whose branch is on origin, naming `lyx adopt <slug>`.
   - `[slug]` given together with `--all`.
   - Evidence: `TestStart_RefusesBeforeTheReconcile` reaches only the task-pair and `lyx beam create` refusals, and `TestStartBeams` covers neither. `PATTERN-refusal-way-forward` asks for a test that reaches each refusal.

3. MEDIUM — D9's failed re-check: the plan never says what happens when the `Beam-Await` status write itself fails, and no row reaches it.
   - That write is what lifts the hold. The record says writes that cross the hold are Beam-Await's, "each retried at the next wake until it succeeds".
   - Card 15 states the retry only for the entry write and the Parent-Align write. `TestBeamAwait` has rows for those two and none for this one.
   - If it is left unretried, the hold stands after a refused landing, so `lyx beam start <slug>`, which the failure notice names, would refuse.

4. MEDIUM — D6 `--all`: no row tests that `--all` names root and every open beam project.
   - Card 4 changes `namedEntries`' `--all` branch, which used to name started tasks, but `TestAdopt`'s planned rows have no `--all` row.

5. MEDIUM — D5 views: no row checks that the list and get views show the effective owner.
   - Card 3 edits `internal/boardengine#Store.ListTasksBrief`, but none of its tests asserts that a task with a stray recorded owner shows its beam's or root's owner. `TestListTasksBriefLayerAndProposal` is not on the card.
   - `renderProjects` showing a beam's effective owner on a board with no prime owner is also untested; only the Running Owner column gets a golden row.

6. MEDIUM — D4 `lyx beam status` shows both parts: only the never-started orch is asserted.
   - Card 11's `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` checks no strand, no watcher and no session.
   - No row checks a recorded session id or a live watcher, so a wrong path in `readOrchView` passes.

7. MEDIUM — the acceptance criterion "no way forward, stencil or skill names" `lyx orch start`, `lyx orch cleanup`, `lyx prime`, `lyx beam promote` or `lyx adopt --prime` is argued, not tested.
   - Cards 1, 10, 12, 25 and 26 each reword sites, but no card adds a scan over the production `.go`, stencil and skill text.
   - `internal/orchengine/launch.go:1`'s file comment names `lyx orch start`, and no card lists that file comment; card 10 only lists `ChooseStartPrompt`.

8. MEDIUM — D11 Worktree-Create of a beam project whose parent beam's pair is absent here: card 18 states the refusal (`lyx beam create <parent>` or `lyx adopt <parent>`) but plans no row.
   - Evidence: card 18's new rows cover the made pair, the member hold and the anchor only.
   - The old `project && !present` branch at `internal/battencli/wire.go:885-891` goes, so this refusal is new behavior for a project.

9. LOW — card 7 (D1): three new `EnsureLive` branches have no `TestEnsureLive` row.
   - A named beam that is no local beam pair, refused naming `lyx beam create <slug>`.
   - A `LandingHeld` read error, which should be that worktree's failure.
   - A nil `LandingHeld`, which should read as no hold.

10. LOW — card 5 run lock: `TestRunLock`'s new rows refuse `project`, `beam`, removal and merge, but no row asserts that changing `kind` or `recipe` of a beam at Beam-Await is refused, which D9 names.

11. LOW — card 21 (D10): two `createSteps` branches have no `TestCreate` row.
    - An owner write lost to another machine, refused naming `lyx adopt <slug>`.
    - The root claim when no root owner is recorded and another machine wins it.

12. LOW — card 25 fallbacks: no row reaches the fall-back to the slug when the `AdoptSubject` seam is nil or fails (dobbyengine), or when a reader leaves the subject empty (`OriginReport.Summary`, battenshed's verdict).

13. LOW — card 15: `BeamAwait` with a nil `Watch` (the timer alone) and a cancelled context returning its error are both stated branches, and neither is a `TestBeamAwait` row.

Additional tool calls:
- I grepped `internal/shedengine` for a `recipe` JSON field and the `internal/boardengine/runlock_test.go` test names. My context lacked both: whether a status file records the recipe D8 says reads as `pawl` (it does not), and whether `TestRunLock` exists (it does).
