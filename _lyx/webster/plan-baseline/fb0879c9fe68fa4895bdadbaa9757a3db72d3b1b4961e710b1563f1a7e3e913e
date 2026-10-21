# Card 3 — board-replay-seam

**Create:**
- `internal/fabricengine#BoardEntry`
- `internal/fabricengine#BoardReplayRequest`
- `internal/fabricengine#BoardReplayResult`
- `internal/fabricengine#BoardReplay`
- `internal/fabricengine#InstallBoardReplay`
- `internal/boardengine#FabricReplay`
- `internal/boardengine#FabricReplay.Entries`
- `internal/boardengine#FabricReplay.Apply`
- `internal/boardengine#TestFabricReplay`
- `internal/fabricengine#ErrBoardReplayMissing`
- `internal/testkit/boardkit#InstallReplay`

**Edit:**
- `cmd/lyx#newRoot`

**Uses:**
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine#LoadConfig`
- `internal/boardengine#RenderToDisk`

**Intent:** D1's Bolt entry merge seam and D5's by-entry Bolt replay, the boardengine entry point beside `boardCriticalSection`.
fabricengine declares the seam it reaches boardengine through, since boardengine imports fabricengine:
`BoardEntry` is one entry as the ended-set rule reads it (slug, status, `created` stamp);
`BoardReplay` has `Entries(boardJSON []byte) ([]BoardEntry, error)` and `Apply(BoardReplayRequest) (BoardReplayResult, error)`;
`BoardReplayRequest` carries the board directory (checked out at origin's tip), the merge base's, the local commit's parent's and the local commit's own `board.json` bytes, and the design files the commit touches with their before and after bytes;
`BoardReplayResult` reports `Empty` (no entry-level or design-file change survives), the values lost to origin and the refusals, each naming the entry, field and reason.
`InstallBoardReplay` stores the implementation process-wide, and the root pre-run built in `cmd/lyx#newRoot` installs `boardengine.FabricReplay{}` beside card 2's install, so the detached push child, Bolt's pull-first and the reconcile all reach it (see Shared Decisions).
A fabricengine read of the seam with nothing installed returns `ErrBoardReplayMissing`, a fail-loud wiring guard in `refusal-spec.md`'s sense naming the root install, which takes no reaching test.
Test binaries never run the root pre-run, so `boardkit.InstallReplay` installs `boardengine.FabricReplay{}` once per process (a `sync.Once`, never restored, so parallel tests share one install of the same value) and every test that reaches a Bolt replay or the ended-set rule calls it; `boardkit` already imports both `boardengine` and `fabricengine`, and only external test packages call it.

`FabricReplay` is a zero value, so `Apply` takes the board config from the request: it loads it with `LoadConfig(configengine.Dirs{Board: <request's board directory>}, "board")`, as `OpenHub` does, applying no skip env, since the replay renders and never syncs.
`FabricReplay.Apply` computes the commit's entry-level changes against its parent (each entry added, removed or with a changed field, and each design file touched), loads a `Store` from origin's `board.json` in the request's board directory, its vocabulary set from that config's `Vocabulary()` as `boardCriticalSection` sets it, and applies each change on its own, against the board as the changes kept before it left it.
Each change passes the board's own checks, `validateWrite` and `checkRunLocks`, never bypassing the run lock; an added entry takes a fresh id from origin's `board.json`.
Where both sides changed the same field of the same entry, or the same design file, since the merge base, or the change touches an entry origin removed, or a check refuses it, origin's side is kept for that change alone and it is reported lost or refused; a later change depending on a refused one (a `depends_on` naming an entry the refused change added) is refused too.
It saves and re-renders `README.md` and the design files with `RenderToDisk` over that config's `Outputs()` when anything was applied, takes neither `board.lock` nor `.gitrepo-push.lock` (its caller holds both) and spawns no sync, so it never calls `boardCriticalSection`.
The refusal classes are those the record lists: the run lock, a dangling or cyclic `depends_on`, the isolated and kind rules, an invalid issue number and an unknown label.

`TestFabricReplay` (untagged, temp board dirs) rows: two different entries changed on the two sides both land and `README.md` is re-rendered; the same field of one entry changed on both sides keeps origin's value and reports the lost local value; a body edit of an entry origin holds under a run status is refused by the run lock and reported, and an unrelated change of the same commit lands; a `depends_on` naming an entry a refused change added is refused too; one table row per remaining refusal class, each refused and reported while an unrelated change of the same commit lands: a dangling `depends_on`, a cyclic `depends_on`, the isolated rule, the kind rule, an invalid issue number and an unknown label, and beside it a row whose label is known only through the board directory's `board.yaml` lands, so the vocabulary is the board config's; the same design file changed on both sides keeps origin's file and reports the local one lost; an added entry takes origin's next id; a change to an entry origin removed is reported lost; a commit whose only change origin already holds reports `Empty` and writes nothing; `Entries` returns each entry's slug, status and `created` stamp.

**ImpactSummary:** Adds one boardengine type and one root install line; no existing board write changes, and no fabricengine caller uses the seam until card 4.

**Commit:** 3: boardengine: the by-entry Bolt replay behind fabric's board seam
