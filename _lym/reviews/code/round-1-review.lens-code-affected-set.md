```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, I looked for what reads, calls or asserts it outside the touched lines.

**List, per change:**

- **`PromoteTo` signature, now returning `PromoteResult`.**
  - Production callers are `internal/boardcli/cli.go:225` and `Board.Promote` (`board.go:441`), both updated.
  - Test callers in `raddlefile_test.go:56`, `slugrule_test.go:226` and `255`, `siblings_test.go:54` and `sketch_test.go:239`, `329`, `337` and `389` are updated.
  - Covered.
- **Promote envelope, now built inline.** `outputSuccessWithTask` (`cli.go:1363`) still serves its other callers, and the inline envelope keeps `task` and `notice`. Covered.
- **`RecordBodySHA256` stamp, three writers.** `StoreDecisionRecord`, `UpsertTaskWithRecord` and `fileRaddleEntry`/`convertRunEntry` all stamp. `scopeChanged` (`runlock.go`) and `UpsertPayloadKeys` docs are updated. Other places that carry or copy the entry's `discussed` flag are not covered; see F1 and F2.
- **`CheckRecordSections` extraction.** `Validate` is the only prior caller and was rewired. The package docs are updated. Covered.
- **`weftLiveness` and `probesOrigins`.** `Add` is rewired and `RecordsBranchLive` is new. No other caller of `probeWeftLeftover` was affected. Covered.
- **PATTERN.md, `pattern/` and stencils.** The diff has no declared PATTERN changes, and no stencil changed. No `Pattern-Departure:` trailer is present or needed, so there is nothing to flag. Covered.
- **Docs for the new flag and warning.** Not covered; see F3.

**Findings:**

- **F1. MEDIUM, design, `internal/boardengine/replay.go:~215`.**
  - `replayFields` lists `discussed` but has no `record_body_sha256` row.
  - The new key is now a known field, so a replay merges it by neither route. The table comment says unknown keys stay as origin holds them.
  - The new `boardengine/doc.go` claims a board sync's replay carries the stamp with the entry, and D5 says the same.
  - A stamp written locally by `--record-file` or `StoreDecisionRecord` is therefore not carried onto origin's entry by a replay.
  - A promote on another machine then reads a stale or missing stamp.
  - Fix: add a `record_body_sha256` row beside `discussed`, with Get and Set on `RecordBodySHA256`, and cover it with a replay test.
  - Evidence: `replay.go:187-216`, `doc.go:551-553`.
- **F2. LOW, scope, `internal/boardengine/store.go:1183-1196`.**
  - The merge path keeps `Discussed` on an entry it removes and upserts again (`keepDiscussed`), but not the stamp.
  - After such a merge the entry has a record and `discussed`, but no `record_body_sha256`.
  - Promote then gives no warning for a body that has since changed.
  - Fix: carry `RecordBodySHA256` in the same branch.
  - Evidence: `store.go:1183-1196`.
- **F3. LOW, consistency, `plugins/ly/skills/board/SKILL.md:94-125`.**
  - The board skill documents `upsert --body-file` and promote, but not `upsert --record-file` or promote's new `warning` key.
  - This is the agent-facing description of the verb and was not updated in the diff.
  - Fix: add a short `--record-file` paragraph and a note on the promote warning.
  - Evidence: `SKILL.md:94-125`, which has no `record-file` match.

**Tool calls (3 of 6):**
1. A combined grep for `PromoteTo` callers, `Only StoreDecisionRecord` text, `RecordBodySHA256` readers, `PATTERN.md` entries and docs that mention upsert or promote. My context lacked the callers outside the diff.
2. A grep and sed for `outputSuccessWithTask`, `SKILL.md` mentions, other `Discussed` readers and the `task.go` comment. My context lacked these call sites.
3. A read of `replay.go:175-260` and `store.go:1175-1202`, found through the `Discussed` grep, to check how replay and merge treat the stamp.
