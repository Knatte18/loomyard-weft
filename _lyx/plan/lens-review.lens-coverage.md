```yaml
lens: coverage
outcome: reported
```

Coverage lens: every decision, acceptance criterion and new branch in the plan, matched to the card and test row that reaches it. Gaps only.

1. MEDIUM — AC "A continue pass… re-exec owes no full pass" / D4 (sleep and start owe a full pass). Card 32's row only records a sleep in the state. Card 33's `TestPlanPass` row "owed gives full" starts from a full pass already owed. No row shows that a dobby start, or a recorded sleep, makes every driven run owed a full pass. No row shows that a re-exec keeps owed passes and owes no new one. Evidence: `plan:internal/dobbyengine#TestRun_LockOwnerAndWakeups`, `plan:internal/dobbyengine#TestPlanPass`.

2. MEDIUM — AC "A full pass owed… stays owed until a full pass… exits 0 with no origin-owed marker standing". `TestDrivePasses_ExitHandling` has no row where a full pass exits 0 while the marker stands and the pass stays owed. It has no row where a failed full pass is retried as a full pass. Its refused-continue row does not assert that no failure is counted. Evidence: `plan:internal/dobbyengine#TestDrivePasses_ExitHandling` (card 33).

3. MEDIUM — AC "counts the run in neither `max_runs` nor its backoff". No row in `TestStartReady_RefusalsAndBackoff` leaves an owned-elsewhere run out of the `max_runs` count. No row in `TestNextWait` or card 33 covers the backoff rule: in flight only while no pause covers the run, and a dobby holding only halted runs backs off (AC "halted runs only included", D13). Evidence: cards 33, 34, 35; `plan:internal/dobbyengine#TestNextWait`.

4. MEDIUM — AC "a driverless running child is respawned… also when each respawned driver lives a while before it dies". `TestInnerRun_RespawnCapStopsWithoutProgress` only drives passes with no driver. No pass sees a live strand or held lock between respawns and checks that the count is not reset. Evidence: card 15.

5. MEDIUM — D12 and AC "Under a covering pause no run enters Land". No row shows a done child under a covering pause yields rather than returning Done. The arm hold is reworked on card 13, but its tests list no covered done row. AC "…or as soon as its pass in flight exits" has no row in `TestPlanPass` either. Evidence: cards 13, 33.

6. MEDIUM — AC "a restarted dobby runs a pass of every run in flight, a run Land already pruned included". No row passes a run whose board entry is gone, driven from its status file alone. `TestDrivePasses_ExitHandling` covers only "a run dropped from the listing is no longer passed". No row makes a held driver lock, rather than a pass dobby itself spawned, read as in flight. Evidence: card 33.

7. MEDIUM — AC on draft: "a note or a project upserted to kind task" gets draft. `TestDraftStatus` has the note→task row and no project→task row. Evidence: card 5.

8. MEDIUM — D2 "`lyx adopt` of a beam, the prime included, refuses while a member… records another machine". `TestAdoptRefusal` has a member row for a beam only, none for `--prime` with a prime member owned elsewhere. Evidence: card 39.

9. MEDIUM — D4 "A hand-run `lyx batten run <slug>` runs one pass… then starts the parent's dobby when it is not live". `ensureParentDobby`, and `StartFirstPass`'s success path (detached spawn, then dobby ensured), are reached by no row. `TestStartFirstPass_RefusesBeforeSpawning` covers refusals only. Evidence: card 28.

10. MEDIUM — Card 29 adds a refusal: "a failed write refuses the resume with its cause and the re-run". No row reaches it. Evidence: `plan:internal/battencli#TestOriginOwedMarker`.

11. MEDIUM — Card 41: `ReadConfigChanges` gains a `git show` contents read, with a nil value for a file absent at a revision. No `fabricengine` test row reaches it, and `TestChangedKeys` only gets bytes handed to it. Evidence: `internal/fabricengine#ReadConfigChanges` on card 41, which plans no test of it.

12. MEDIUM — Card 6's new branch where `turnIntoProject` clears a draft status has no test row. Evidence: `internal/boardengine#turnIntoProject` on card 6; no SetOwners or promote row is planned.

13. MEDIUM — AC "A pair teardown of a landed beam ends its dobby". No row: card 37 registers `RoleDobby` under the beam slug untested, and card 40 tests only the session-down spare. Evidence: cards 37, 40.

14. LOW — Constraint `PATTERN-shed-driver` "a yield at Debug": no card says a yield logs at Debug. Cards 8, 10 and 13 carry the yield without its log level.

15. LOW — Constraint "it starts `lyx batten run` processes, registered in `internal/runproc`". Card 33's `SpawnPass` says "registered nowhere". The constraint still holds through batten's own `registerSeededRun`, but no card says so. Evidence: card 33.

16. LOW — AC `--continue-pass` "not `running` at Run-Shed": the rows cover "a run at Land", but none covers a run halted at Run-Shed. Evidence: `plan:internal/battencli#TestBattenPreRun_ContinuePass`.

17. LOW — D7: the refused-start notice "cleared when a pass claims the task or the task's board entry changes" has no row. "one notice per refusal text" has no row with two texts. Evidence: card 35.

18. LOW — AC "`lyx orch start`, with or without `--continue`, calls it first" has no `--continue` row. Evidence: `plan:internal/orchcli#TestStart_StartsTheDobbiesFirst`.

19. LOW — Several new branches have no row:
    - card 27: `DobbyGeometry`'s absent-worktree error;
    - card 31: `dobbyReading`'s read error, `ReviveStalled`'s error return, and the `Awake` value across passes;
    - card 25: `battenConsumeAttempt`'s non-numeric attempt;
    - card 32: a tick failure logged once per episode;
    - card 18: a paused Land stop routed to no notice;
    - card 21: a worktree-layer loom value not copied;
    - card 37: `lyx beam start` run from a beam, and `dobbyCmd`'s registration.

20. LOW — AC "A batten stop the tackler escalates reaches the arbiter". Card 25 only asserts that the seam is set and that `EscalationPending` is false. No row shows that a tackler escalation at a batten stop calls `Judge` through `shedarbiter.Seam`. Evidence: `internal/battencli#TestBattenDriverDeps` on card 25.

Further tool calls: none.
