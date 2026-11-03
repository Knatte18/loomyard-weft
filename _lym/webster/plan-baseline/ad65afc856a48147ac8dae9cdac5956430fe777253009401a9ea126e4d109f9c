# Card 1 — board-decision-record

**Create:**
- `plan:internal/boardengine#decisionFilePrefix` -> `const decisionFilePrefix = "decision-"`
- `plan:internal/boardengine#decisionFileSuffix` -> `const decisionFileSuffix = ".md"`
- `plan:internal/boardengine#decisionFileName` -> `func decisionFileName(slug string) string`
- `plan:internal/boardengine#ErrRecordEntryDone` -> `var ErrRecordEntryDone = errors.New("a landed entry keeps the decision record it last stored")`
- `plan:internal/boardengine#Board.StoreDecisionRecord` -> `func (b *Board) StoreDecisionRecord(slug string, record []byte) error`
- `plan:internal/boardengine#Board.DecisionRecord` -> `func (b *Board) DecisionRecord(selector any) ([]byte, bool, error)`
- `plan:internal/boardengine#checkDiscussedPayload` -> `func checkDiscussedPayload(s *Store, fields map[string]any) error`
- `plan:internal/boardengine_test#TestStoreDecisionRecord` -> `func TestStoreDecisionRecord(t *testing.T)`
- `plan:internal/boardengine_test#TestDiscussedPayload` -> `func TestDiscussedPayload(t *testing.T)`

**Edit:**
- `internal/boardengine#Task`
- `internal/boardengine#scopeChanged`
- `internal/boardengine#UpsertPayloadKeys`
- `internal/boardengine#Store.UpsertTask`
- `internal/boardengine#Store.UpsertTasksBatch`
- `internal/boardengine#Store.MergeTasks`
- `internal/boardengine/doc.go`
- `internal/boardcli#storeVerbs`

**Uses:**
- `internal/boardcli#TestPayloadUse_MatchesDeclaredKeys`
- `internal/boardengine#Board.SetRunStatus`
- `internal/boardengine#checkFrontPayload`
- `internal/boardengine#designFilePrefix`
- `internal/boardengine_test#newRenameBoard`
- `internal/boardengine_test#upsertLocked`

**Intent:**
Give a board entry a decision record (D1, D2, D3).

Declare `decisionFilePrefix` and `decisionFileSuffix` in `replay.go`, beside `designFilePrefix` and `designFileSuffix` (`PATTERN-named-constants`).
`decisionFileName(slug)` joins them into `decision-<slug>.md`, the one place the name is formed.
No render output names this file, so `RenderToDisk`'s manifest never holds it and a render never deletes it.

`Task` gains `Discussed bool` with the JSON tag `discussed,omitempty`, so `board.json` stores no key while it is false.
An older binary keeps the key as an unknown one (`PATTERN-tolerant-records`).
`scopeChanged` keeps from comparing it, and its doc comment names `Discussed` among the fields that are not scope, beside RunSettings.
So a run-held entry takes the flag, and the write stamps no revised date.

`Board.StoreDecisionRecord(slug, record)` runs in one `boardCriticalSection`.
- An absent slug is refused: `no board entry %q to store the decision record on: restore the entry with `lym board upsert`, then re-run the step that stored`.
- A done entry is refused with an error that wraps `ErrRecordEntryDone`, so `errors.Is` matches it. Its text names that a landed entry keeps the record it last stored, since the run ledger sets aside a board commit whose entries have all ended.
- When the entry is already `Discussed` and `decisionFileName(slug)` already holds exactly `record`, it returns `noWrite{}`, so nothing is saved or rendered and no sync is spawned.
- Otherwise it writes the file with `fsx.AtomicWrite` under the board directory, inside the mutation and so before `board.json` is saved, then sets `Discussed` on the entry.
- A crash between the two steps leaves a file with no flag, never a flag with no file.

The store bypasses the run lock for the flag and the file only; every other field of a held entry stays locked.
Its callers are the Discussion-Bouncer approve seam, `lym loom decision add` and `lym loom decision store` (cards 7 to 9).

`Board.DecisionRecord(selector)` reads without the lock, as `GetTask` does.
- It returns the file's bytes with found true only when the entry exists, carries `Discussed` and the file exists.
- An absent entry, an unflagged entry and a flagged entry with no file each answer found false.
- A read error other than not-exist is returned.

`checkDiscussedPayload` follows `checkFrontPayload`'s shape.
- A payload `discussed` that is a bool equal to the stored entry's value is deleted from the fields as a no-op. The stored value is false for a new entry.
- Any other value is refused: `entry %q: the discussed flag is never set by hand: lym sets it when a Discussion is approved, and `lym loom decision store` re-copies the record of a run past its approval; drop the discussed key from the payload`.
- `Store.UpsertTask`, `Store.UpsertTasksBatch` and `Store.MergeTasks` call it beside `checkFrontPayload`, and `UpsertPayloadKeys` lists `discussed` among the optional keys, with a doc-comment line saying it is accepted only as the stored value.
- The `upsert` command in `internal/boardcli#storeVerbs` names the key twice. Its `Use` token gains `discussed?` after `on_land?`, since `TestPayloadUse_MatchesDeclaredKeys` fails while that token and `UpsertPayloadKeys` disagree. Its `Long` field list, which names every accepted payload key with `beam`, `front` and `on_land` among them, gains a `"discussed"` line: a bool lym sets, accepted only as the entry's stored value.
- In a merge the check runs before the removals, so a merge that upserts a slug it removes is judged against the removed entry's stored value.

`Store.MergeTasks` remembers the `Discussed` value of a removed entry whose slug the merge upserts again.
After its `UpsertTask` call, it sets that value on the upserted entry, so the rebuilt entry keeps its flag (D4's merge bullet).

`doc.go` gains a `# Decision record` section stating each of the following:
- the file's name and place;
- that only lym writes the flag and the file;
- the run-lock exemption and its bound;
- `DecisionRecord`'s found rule;
- the payload rule.
Its `# Run lock` paragraph names `discussed` beside status as a field a held entry takes.

Tests, both tier 0, in the external `boardengine_test` package, built on `newRenameBoard`'s config shape with a fake clock.

`TestStoreDecisionRecord` is a table with these rows:
- A store lands the bytes in `decision-<slug>.md` byte for byte. `board.json` holds `"discussed":true` and no record text. `Rerender` after it leaves the file in place.
- On an entry whose status is `RunStatus("running","Discussion-Bouncer")`, the test advances the fake clock's `*now` before the store; the store succeeds and `Revised` and every other field stay as loaded.
- A repeat with the same bytes writes nothing. The test deletes the rendered README after the first store and asserts it is still absent after the repeat, since only a write renders.
- A store onto an unflagged entry whose `decision-<slug>.md` already holds the same bytes, written by hand, still sets `Discussed`.
- A repeat with new bytes overwrites the file.
- An absent slug is refused, and the error contains `lym board upsert`.
- A done entry is refused, `errors.Is(err, boardengine.ErrRecordEntryDone)` holds and no file is written.
- `DecisionRecord` answers found false for a `decision-<slug>.md` written by hand on an unflagged entry.
- `DecisionRecord` answers found false for a flagged entry whose file was deleted.
- `DecisionRecord` returns an error for a flagged entry whose `decision-<slug>.md` is a directory, the read error that is not not-exist.

`TestDiscussedPayload` is a table over the upsert, batch and merge paths on an entry flagged by `StoreDecisionRecord`:
- `discussed:false` on a flagged entry is refused naming `lym loom decision store`.
- `discussed:true` on a new entry is refused.
- `discussed:"yes"` is refused.
- The stored value passes and leaves the flag as stored.
- A merge that removes the flagged slug and upserts it again keeps `Discussed` true on the rebuilt entry.

**ImpactSummary:** All changes are inside boardengine. `Task` gains one omitempty field, and the three store payload paths gain one check each. No caller outside the package changes, and existing payloads, which never carry `discussed`, are unaffected.
