# Card 3 — The board's front and on_land fields

**Create:**
- `plan:internal/boardengine#checkFrontPayload` -> `func checkFrontPayload(s *Store, fields map[string]any) error`
- `plan:internal/boardengine#Board.FileFrontTask` -> `func (b *Board) FileFrontTask(fields map[string]any, onLand string) (Task, error)`
- `plan:internal/boardengine_test#TestFrontTask` -> `func TestFrontTask(t *testing.T)`
- `plan:internal/boardcli_test#stepFrontTask` -> `func stepFrontTask(t *testing.T, f *cliFixture)`

**Edit:**
- `internal/boardengine#Task`
- `internal/boardengine#UpsertPayloadKeys`
- `internal/boardengine#Store.UpsertTask`
- `internal/boardengine#Store.UpsertTasksBatch`
- `internal/boardengine#Store.MergeTasks`
- `internal/boardengine#Store.Rename`
- `internal/boardengine#replayFields`
- `internal/boardengine#scopeChanged`
- `internal/boardcli#storeVerbs`
- `internal/boardcli_test#TestCLI`

**Extends:**
- `internal/boardengine/doc.go`
- `internal/boardcli/doc.go`

**Uses:**
- `internal/boardcli_test#seedBoardFile`
- `internal/boardcli_test#runJSON`
- `internal/boardcli_test#runCLI`
- `internal/boardcli_test#cliFixture`

**Intent:**
D8's two board fields and their guards.

`Task` gains `Front bool` (`json:"front,omitempty"`) and `OnLand string` (`json:"on_land,omitempty"`): a task lym files ahead of its worktree's other starts, and the key of the action `internal/fronttask` runs when it lands.
`taskKeys` follows from the tags, so an older binary keeps both keys through `Unknown` (PATTERN-tolerant-records).
`replayFields` registers `front` and `on_land` like `notify_on_land`, so a replay of a local board commit merges them as fields and never drops a filing's fields; `scopeChanged` compares both, as it compares `Kind`, and its doc names them, since they never change after the filing and a run-held entry takes no change to them.

`UpsertPayloadKeys` gains `front` and `on_land`, with the doc line that each is accepted only as the entry's stored value, as `beam` is.

`checkFrontPayload` judges a payload against the entry `fields["slug"]` names, the entry being new when the store holds none, and runs in `Store.UpsertTask`, `Store.UpsertTasksBatch` (for each task, before any is applied) and `Store.MergeTasks`, right after `checkBeamPayload`:
- A `front` or `on_land` value equal to the stored one (false and empty for a new entry) is dropped from fields; any other value, or a `front` that is not a bool or an `on_land` that is not a string, is refused: `entry %q: only lym files a front task and writes its front and on_land fields; way forward: drop the front and on_land keys from the payload`.
- On an entry whose stored `Front` is true, a `project` value other than the stored project, or a `kind` value other than the stored kind, is refused: `entry %q is a front task, and a front task's project, kind and slug are fixed; way forward: remove it with "lym board remove %s", preceded by "lym batten cancel %s --remove" while a run holds the entry`.
  A value equal to the stored one passes untouched.
Reached by `lym board upsert`, `upsert-batch` and `merge` naming an existing front entry, which no earlier check refuses for these keys.

`Store.Rename` refuses an entry whose `Front` is true, with the same fixed-fields text, after its absent-entry and run-held checks; reached by `lym board rename` of a released front task.

`Board.FileFrontTask` is the one write that sets the two fields: under the write lock it upserts the task `fields` describe through the same store path, with `Front` true and `OnLand` set to onLand on the stored entry, skipping only `checkFrontPayload`; the payload itself must carry neither key, and one that does is refused as the upsert paths refuse it.
The board's draft rule makes the new task draft, so a caller releases it with a later `SetStatus(slug, nil)`.
It carries no strand name of its own, so the run-agent task guard judges the Board's configured `StrandName`, which `fronttask` leaves empty.
Bound: only `internal/fronttask` calls it (PATTERN-front-task, written on card 7); a board edit cannot set either field, change a front task's project or kind, or rename it.

`storeVerbs`: upsert's `Use` token gains `front?, on_land?` after `beam?`, and its long help states that both are written only by lym and accepted only as the stored values, and that a front task's project, kind and slug are fixed; `TestPayloadUse_MatchesDeclaredKeys` then matches through `upsertPayload`, which reads `UpsertPayloadKeys`.

Tests:
- `TestFrontTask` (tier0, external package) holds only what no board verb reaches: `FileFrontTask` stores a draft task with `Front` true and `OnLand` set, and a release through `SetStatus(slug, nil)` leaves it released with an empty status; a payload to `FileFrontTask` carrying `front` is refused.
  The payload and fixed-fields refusals are pinned once, through the verbs, by `stepFrontTask`.
- `stepFrontTask`, added to `TestCLI`'s steps after `Owners`, seeds a board.json holding a released front task with `on_land` `adopt-check`, then: `lym board upsert`, `upsert-batch` and `merge` with `"front":false` each exit 1 naming the drop of both keys; each with `"on_land":"x"` the same; `lym board upsert` with `"front":"yes"` and with `"on_land":7` each exit 1 the same; `lym board upsert` echoing `"front":true,"on_land":"adopt-check"` exits 0 and `get` reads both unchanged; `lym board upsert` of a new task with `"front":false,"on_land":""` exits 0 and stores neither; `lym board upsert` of a title change of the front entry exits 0; `lym board upsert` with another `project` and with `"kind":"note"` each exit 1 naming `lym board remove`; `lym board rename` exits 1 naming it; `lym board remove` of the entry exits 0 and leaves no entry.

`boardengine/doc.go` gains the `front` and `on_land` paragraph: what they mean, `FileFrontTask` as their one writer, the payload and fixed-fields refusals.
`boardcli/doc.go` gains the line that upsert takes `front` and `on_land` only as stored values.

**ImpactSummary:** two new Task fields, registered in the replay table and the run-held compare, one new payload check on the three upsert paths and one rename refusal, all in boardengine, and boardcli's upsert help; no entry without the fields changes behaviour.

**Commit:** 3: board: front and on_land fields only lym writes
