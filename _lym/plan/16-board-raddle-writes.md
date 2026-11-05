# Card 16 — board-raddle-writes

**Create:**
- `plan:internal/boardengine#RaddleEntry` -> `type RaddleEntry struct`
- `plan:internal/boardengine#RaddleFiling` -> `type RaddleFiling struct`
- `plan:internal/boardengine#ErrRaddleRecheck` -> `var ErrRaddleRecheck = errors.New("the board moved since the raddle proposal was checked")`
- `plan:internal/boardengine#raddleFiled` -> `type raddleFiled struct`
- `plan:internal/boardengine#Board.FileRaddle` -> `func (b *Board) FileRaddle(f RaddleFiling) error`
- `plan:internal/boardengine#Board.ReleaseTasks` -> `func (b *Board) ReleaseTasks(slugs []string) error`
- `plan:internal/boardengine#Board.EndRaddleEntry` -> `func (b *Board) EndRaddleEntry(slug string) error`
- `plan:internal/boardengine#TestFileRaddle` -> `func TestFileRaddle(t *testing.T)`
- `plan:internal/boardengine#TestReleaseTasks` -> `func TestReleaseTasks(t *testing.T)`
- `plan:internal/boardengine#TestEndRaddleEntry` -> `func TestEndRaddleEntry(t *testing.T)`

**Edit:**
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine/doc.go`

**Uses:**
- `internal/boardengine#applyDraftRule`
- `plan:internal/boardengine#RaddleStoreDir`
- `internal/boardengine#DecisionFileName`
- `internal/boardengine#Board.StoreDecisionRecord`
- `internal/boardengine#Board.FileFrontTask`
- `internal/boardengine_test#TestDraftStatus`

**Intent:** D9 steps 3, 5 and 7's board writes, each one `boardengine` write under the board lock (Fact: `StoreDecisionRecord` refuses an absent entry, so filing needs a write that creates its entries with their records).
`RaddleEntry` is one filed entry: slug, title, kind, labels (its type label and the Sketch's module labels), recipe, profile, `depends_on`, body and its decision record bytes.
`RaddleFiling` carries the Sketch slug and its `created` stamp, `Project` (the Sketch's own `project` field, the project level D9 files at, empty for the top level), the approved proposal's hash, the run's entry slug, the project entry when the shape files one, the parts, whether the parts are released, the staged store files (`proposal.yaml`, `cold-proposal.yaml`, `decision-record.md`) and `Check func(tasks []Task, store RaddleStore) error`, D8's re-check, run inside the write.
Filing at the Sketch's own `project` keeps each filed entry's parent beam, which `ParentBeam` reads over the `project` links, equal to the Sketch's own, with no other parent read, as the decision record's Added after Discussion entry "Filed entries take the Sketch's own project field" settles in place of the `PATTERN-pair-parent` Constraint's `hubgeom` read.
`FileRaddle` writes nothing when the store already holds a directory for the Sketch slug whose `created` and `hash` equal the filing's (the step's done marker); otherwise, in one write, it:
- runs `Check` over the loaded entries and the store as they stood, returning its refusal wrapped in `ErrRaddleRecheck`, so a caller tells the re-check's refusal from a failure of the write;
- adds the project (its type label the proposal's project `type`, the Sketch's module labels, no `sketch` label, discussed) with its `project` field set to `Project`, and each part as a discussed task, its `project` set to `Project` or to the new project, with its recipe, profile, labels, `depends_on` (a slug gone from the board dropped) and body;
- writes each filed entry's `decision-<slug>.md` before saving, with its `discussed` flag set in the same write, as `StoreDecisionRecord` orders the file before the flag;
- writes the store directory with `store.json` recording the hash, the `created` stamp and every filed slug, the project's included, so the write leaves its own done marker, and the board sync commits the store with the entries (Fact: a board write spawns the detached sync, whose commit stages every change in the Bolt);
- leaves the run's entry open.
It returns a `raddleFiled` result naming the parts to leave released; `boardCriticalSection` passes those slugs as card 13's `applyDraftRule` `released` list, so a released filing is released in its own write and an unreleased one (the `beam` shape) is draft.
The writer is Go with no strand name, so `checkTaskCreation` still refuses a run agent's own `lym board upsert` creating a task.
`ReleaseTasks` releases, in one write, each named task still on the board, open and draft, and skips one absent or done; it writes nothing when none is left to release, which is step 5's done marker.
`EndRaddleEntry` marks the entry done and clears its `notify_on_land` in one write, which the run lock admits (Fact: `scopeChanged` compares neither `Status` nor `NotifyOnLand`); an absent or done entry writes nothing.
`TestFileRaddle` (tier0) has rows: each shape (`tapestry` and `tasks` released at the Sketch's level; `project` filing the project and its parts released; `beam` filing them draft; in the `project` and `beam` rows the project carries the proposal's type label), each filed entry discussed with its record file in the same write; a Sketch inside a grouping project that is no beam (parts and project carry that project); the store directory and `store.json` written with every filed slug, the project's included; a second call with the same hash writing nothing; a `Check` refusal writing nothing and matching `ErrRaddleRecheck`; a `depends_on` on a slug gone from the board dropped.
`TestReleaseTasks` (tier0) has rows: two draft parts released; one absent and one done skipped; none left (no write).
`TestEndRaddleEntry` (tier0) has rows: an open entry with `notify_on_land` (done, field cleared, one write); an absent entry and a done one (no write).
`TestDraftStatus` covers the draft rule for every other new task as it stands.

**ImpactSummary:** Three new board writes reaching the draft rule's released list only through a `raddleFiled` result; every existing write keeps the draft rule as it is.
