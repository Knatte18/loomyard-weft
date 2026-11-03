```yaml
lens: affected-set
outcome: reported
```

affected-set lens: for each change a card makes, find everything that reads or asserts it, and check which cards cover that set.

**Per change: the affected set and the cards that cover it**

1. **`render.Display` gains `Order`; JSON decode/encode of `order`** (card 1)
   - Set: `render/types.go` `Display`, `types_test.go` (both tests), reed.json persistence through `Display`.
   - Covered by card 1.
2. **Overview placement by order key** (card 1)
   - Set: `overviewWindowInsertArgs`, its caller `openOwnWindowLocked`, `TestWindowOrder`, `TestWindowOrder_RealTmux`, `windoworder.go` header, `doc.go` "Window order" paragraph.
   - Covered by card 1. `bindings_integration_test.go:234,240` expects the Alt wrap to land on the window named `overview`; that title doesn't change, so it is unaffected.
3. **Overview cap removed** (card 1)
   - Set: `addStrandLocked` loop, `TestAddStrandLocked` SecondOverview, the `ReplaceStrand` doc comment (`replace.go:41`), `doc.go:428,431-432`, `render/types.go:24` (`AnchorOverview` comment), the `isOverview` doc comment (`ownwindow.go:170`, "the session's overview window").
   - Card 1 covers all of it except the `isOverview` doc comment.
4. **`UpdateStrand` order refusal** (card 1)
   - Set: `refuseWindowMove`, `TestUpdateStrandLocked`, `doc.go:430`.
   - Covered by card 1. There are no production `UpdateStrand` callers.
5. **Focus fallback takes the lowest-index overview** (card 2)
   - Set: `agentWindowPanes`, `relevantWindow`, `followTargetWindow`, the doc comments of `FollowRun`, `selectRelevantWindowIfCurrentLocked`, `reconcileLocked` and `killStrandPanes`, `doc.go:81,413,419`, the attach and follow tests.
   - Also in the set: `reedcli/follow.go:38`, the user-facing `lym reed follow` help ("the newest live agent window, else the overview").
   - Card 2 covers everything except `reedcli/follow.go:38`.
6. **Runs strand no longer replaces the loom status strand** (card 3)
   - Set: `ensureRunsStrand`, `RunsStrandName` comment, `LoomStatusStrandName` comment (`loomengine/driver.go:27`), `orchcli/doc.go:13`, `orchengine/doc.go:14`, `TestStart_EnsuresTheRunsStrandBeforeTheOrch`, `docs/overview.md:687` (batten entry) and `docs/overview.md:424` (orch entry: "ensuring it replaces a `loom-status` strand a pawl run's `lym loom start` put in the overview window first").
   - Card 3 covers all but `docs/overview.md:424`. It edits `docs/overview.md` but its Intent names only the batten entry's sentence.
7. **Runs strand holds the first overview window, order zero** (card 3)
   - Set: `runsStrandAddSpec` and its doc comment (`runs.go:39`, "the session's overview window"), the `runs.go:1` file header (same phrase), `orchcli/doc.go:13`, `orchengine/doc.go:14`.
   - Card 3 edits `runsStrandAddSpec`'s code and the two doc.go files. Its Intent leaves `runs.go:1` and `runs.go:39` unchanged.
8. **Status strand always takes the overview, titled with its recipe, and loses the strands parameter** (card 4)
   - Set: `statusStrandAddSpec` and its doc comment, its caller `ensureStatusStrand`, `TestStatusStrandAddSpec`, the two `smoke_drivingsurface_test.go` calls (126, 229), the subtest "start beside a runs strand…", that file's header (line 5), `loomcli/doc.go:62`, `docs/overview.md:544,821`.
   - Covered by card 4.
   - The task run's checklist window is now titled `tapestry` instead of `overview`. I searched for anything asserting the old name: no loomcli smoke test asserts the status window's name or index (`smoke_driverstrand_test.go`, `smoke_starttail_test.go` and `smoke_test.go` have no overview or window-name assertion). Only the reed test helpers key on `overviewWindowName`, and they add their own strands.

**Findings**

- **F1 — MEDIUM, consistency, card 3 (`docs/overview.md`).** Card 3 covers part of the docs set D4 falsifies. Its Intent rewrites only the batten entry's sentence (`docs/overview.md:687`). The orch entry at `docs/overview.md:424` still says "ensuring it replaces a `loom-status` strand a pawl run's `lym loom start` put in the overview window first", which D4 makes false. Missed member: `docs/overview.md:424`. The file is already a card 3 target, so the fix is an Intent addition only.
- **F2 — LOW, scope, cards 1, 2 and 3 (method).** The scope line asks for every reed, orchcli and loomcli comment that states one overview per session. The cards cover that sweep only in part. Missed members:
  - `internal/reedengine/ownwindow.go:170`: the `isOverview` doc comment, "places a strand in the session's overview window". Card 1 has `isOverview` only under Uses.
  - `internal/orchcli/runs.go:1`: the file header, "the session's overview window".
  - `internal/orchcli/runs.go:39`: the `runsStrandAddSpec` doc comment, "the session's overview window". Card 3 edits this member but says nothing of its comment.
  - `internal/reedcli/follow.go:38`: the `lym reed follow` help, "else the overview". It now needs "else the lowest-index overview" under D2, and no card lists `reedcli`.

Tool calls:
1. Grep for `overviewWindowName` / `"overview"` and for the files referencing the loom-status strand: to find assertions on the old checklist window name. The manifest lacked them.
2. Grep of those status-strand files for overview/window assumptions: to check whether other smoke or batten tests depend on the status window's placement. Not in the manifest.
3. Grep of the loomcli smoke tests' window assertions and of docs for overview mentions: to confirm no other test asserts the status window, and to find the `docs/overview.md:424` orch entry text.
4. Grep of the docs reed entry and the batten and orchengine overview comments: to check for further one-overview statements beyond the manifest's reads.
