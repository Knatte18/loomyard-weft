```yaml
lens: guard-reach
outcome: reported
```

The guard-reach review found two MEDIUM, four LOW and no BLOCKING defects.

**MEDIUM**

1. **Card 12, D6/D8 — `depends-unknown`: a dependency on a pruned `board_checked` entry passes only when `Filing` is true.**
   - At `raddle-approve` the D6 check runs with `filing` false (card 20 uses `CheckRaddleProposal(check, false)`). A dependency on a weighed entry that has since been pruned therefore fails as `depends-unknown`.
   - In the same call, D8's re-check (`RecheckBoard`) counts that entry as landed and names it without failing. The approve is refused, and the agent must drop a dependency D6 admits: "names only parts, entries on the board, or board_checked entries since gone from it, which D8 counts as landed", with no restriction to filing.
   - Evidence: card 12's `depends-unknown` bullet ("with `Filing` true a `board_checked` entry gone from the board passes as landed"), card 20's propose and approve rows, and decision record D6 and D8.
   - Fix: admit a gone `board_checked` dependency whatever `Filing` says, and add a `TestCheck` row with `Filing` false.

2. **Cards 3 and 5, D12 — the session-end hook is installed only on loom's run runner and the card-fixer runner, but sessions are also stopped through other runners.**
   - `Runner.stopRecorded` fires the hook only on the runner that holds it. A session ended by one of these runners writes no usage file:
     - the watchdog's step stop: `internal/loomcli/looparm.go:122` builds a fresh runner, then `StopStepRuns` calls `stopRecorded` (`internal/shuttleengine/stepstop.go:86`);
     - `lym batten cancel` (`internal/battencli/cancelcmd.go:276`);
     - pair teardown's cancel (`internal/pairteardown/cancel.go:259`).
   - D12 asks for a file "at every session end the provider can read, whatever the outcome". A killed or cancelled session is exactly that outcome, and its run's cost cell then undercounts.
   - Card 3's row "a StopStrand of a running unawaited record calls it once" passes only on a hooked runner.
   - Evidence: `internal/shuttleengine/stop.go:62-91`, plus the three sites above.
   - Fix: install `UsageRecorder` on those runners where the location is a run's worktree, or record the stop from the waiting run's own finalize.

**LOW**

3. **Card 13, D2 — the test row "plain promote of a Sketch (refused with the way forward)" never reaches `checkDiscussedPromotion`.**
   - A Sketch carries `sketch` as its only type label. `promoteTo` calls `validateWrite`, which reaches card 13's new `validateLabels` rule ("sketch on a task whose recipe is not raddle") inside `fn`. That refuses the write before `boardCriticalSection` runs `checkDiscussedPromotion`.
   - D2's refusal text is reached only by a discussed note whose `sketch` label a note write stripped. The row as written asserts the wrong refusal.
   - Evidence: `internal/boardengine/labels.go:14-43` (`validateLabels` runs in `validateWrite`), `internal/boardengine/board.go:111-157` (`fn` runs before the checks).
   - Fix: the row should name which refusal it expects, and a row with the stripped label should reach `checkDiscussedPromotion` through a plain promote.

4. **Card 14, D10 — `checkNewSlugs`' new store refusal is not run on the replay path.**
   - `replayRun.addEntry` adds an entry through `normalizeTask` and `validateWrite` only, never `checkNewSlugs`. A board commit from a machine that has not yet pulled the store can land a task under a filed slug on origin.
   - Card 24 cites only this route as the reaching input for its "store holds the slug" start refusal. The card does not state that the replay is unguarded or bound it.
   - Evidence: the board subagent report on `replay.go` `addEntry` ("It does not run `checkNewSlugs`").

5. **Cards 14 and 25, D10/D11 — `store.json` records `project` apart from `filed`, and no card says whether `filed` holds the project slug.**
   - `Holds` answers "the Sketch whose `filed` holds the slug". If the project slug sits only under `project`, three things built on that lookup are unreachable for a project or beam shape:
     - card 14's row "new project of the pruned filed project's slug (refused)";
     - card 25's `raddleHandOff` lookup by the beam project's slug;
     - card 17's lookup by the project slug.
   - Evidence: card 14's `store.json` bullet and card 16's "the filed slugs".

6. **Card 6, D12 — a resumed Finalize records no landed size.**
   - `RecordLandedSize` takes the pre-merge parent tip from a `Finalize` field set inside `mergeParent`. A Finalize resumed in a new process after its merge step has no field value, so the skip branch fires and the run loses its size, entering the cost fit's means only.
   - Evidence: `internal/landingshed/finalize.go:1012-1022` (`mergeParent` reads `readParentMergeBefore` in-process), and `recordLanding` at lines 341 and 364.
   - Fix: persist the tip, or read the size from the parent's first-parent range.

Outside this lens: card 16 files parts at `hubgeom.ParentOf`'s nearest open beam. That drops the Sketch's own grouping project (`Task.Project`) when the Sketch sits in a non-beam project, which D9's "Sketch's own project level" does not do.

Tool calls:
- Read `internal/boardengine/labels.go` and the pawl and darn row engines: to know where the label rule refuses and whether pawl and darn hold Record-Write.
- Read `internal/loomcli/looparm.go:105-140` and the `stopRecorded`/`StopStrand` locations: to check which runners stop sessions.
- Read `internal/shuttleengine/stop.go:55-100` and the stop call sites in batten and pairteardown: to confirm those stops bypass the hook.
- Grepped `internal/boardengine/store.go` for kind changes: to confirm an upsert can make a note a task.
- Grepped `internal/landingshed/finalize.go` for the landing paths: to trace where the pre-merge tip is read.
