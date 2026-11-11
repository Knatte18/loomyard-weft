# Support log — decision-record-replace

## Interview

### Turn 1 — board and exploration (autonomous)

The run is autonomous, so the operator answers no questions.
The board entry folds issues #114, #116 and #117 into one task.
No `Decision for decision-record-replace:` note exists.
Touching entries:

- `discussed-task-split` (F14) holds the history of the stored record, Record-Write seeding and raddle.
- `parent-reviews-decision-record` (F5) is about a parent review before Plan and does not overlap.
- `raddle-dep-and-loop-exit` (T7) changes a different check of `raddle-propose`, so it does not conflict.

`lym loom peers` lists no parent commits, and no peer's changed paths touch the decision verbs or the board record.

Three explore subagents and my own reads established these facts:

- `lym loom decision add` appends an entry with no id. It commits, copies the record to the board only past Discussion approval, and posts each live strand a notice.
- The dropped-entry history check (`discussionparser.CheckAddedDecisionsKept`) compares the record with every committed version that `fabricengine.CommittedAnchoredFileHistory` returns, so no writer can remove an added entry.
- Nothing folds added entries.
- The board keeps the record as `decision-<slug>.md` beside `board.json`.
  `boardengine.Board.StoreDecisionRecord` writes it, called from Discussion approval and `decision store`, and `upsert` cannot replace it.
- Raddle's Record-Write seeds the run's record from the board. Its gate and `raddle-propose` validate against that file through `internal/discussionparser`, and the body is never read.
- `IdentityText` drops added entries and carry-over blocks from the review subject.

### Turn 2 — design picks (autonomous)

Every pick below is an auto-pick; see the question ledger.

- A whole-record replace (D1) with a base-hash guard.
- A commit trailer that bounds the history check (D2).
- A board copy for every run whose entry carries a stored record (D3).
- `upsert --record-file` (D4).
- A promote warning (D5).
- Guidance in the parent directive and the spec (D6).

### Turn 3 — lens review

All six forks returned a report.
Duplicates are merged: one line per finding, with its lens and its outcome.

- interactions 1, bound-breaker 1 and 2 (BLOCKING): a failed or swept commit could land the replaced bytes without the trailer, and `lym fabric commit` cannot carry one.
  Fixed. D1 now commits through `fabricengine.CommitAnchoredPathsWithTrailers`, makes the trailer commit even when the bytes already landed, and on failure restores the record and names `replace`.
  D2's trailer now carries the sha256 of the written bytes, and the cut is the newest version that matches it.
  On the "no lym path can carry a trailer" claim, the existing trailer commit seam proves it wrong; the fix still applies.
- interactions 2, existing-paths 3: `--amends-plan` roll-back keys on added headings, and amendment titles have readers. Fixed: replace takes no `--amends-plan`; D1 says to amend with `add --amends-plan` and fold later.
- interactions 3, existing-paths 1 and 2, walkthrough 1, 3 and 4, bound-breaker 3, 6, 7 and 8: the `revised` and `record_stored` comparison fires on every motif Sketch, misses Raddle-File's writer, depends on clock reads and can be skewed by the payload.
  Fixed: D5 now compares a `record_body_sha256` body hash, stamped by every writer, Raddle-File included, carried by replay as an entry field, and compared before the promote's write.
- interactions 4, board-scope 1 and 2: the fold path for a motif-made Sketch was unnamed, and the departure from #114's fixer fold was stated only in Out. Fixed: D6 names `--record-file` for a Sketch and states the departure with its reason.
- existing-paths 4: a re-seed after a failed board copy. Fixed as an Open risk naming `decision store`; D3 names the same way forward.
- bound-breaker 4: the base check and write were not atomic. Fixed: D1 adds a per-run decision lock over `add`, `replace` and `store`.
- bound-breaker 5: a board replace in the window before a run holds the entry. Fixed: D4 also refuses an entry whose slug has a live batten run.
- walkthrough 2: `Validate` needs a support log the board lacks. Fixed: D4 names a record-only section check.
- leanness 1, 2, 3, 4, 5, 6, 7 and 8: Fixed.
  The D2 rationale no longer names a rejected alternative, and the Out bullets drop their reasons.
  Scope In is cut to one line, and the caller list in D2 is gone.
  The constraints are shortened, and the repeated assumptions and the `board.json` hand-edit risk are cut.
  D1's rationale is trimmed.
- leanness 9: Rejected; the D5 rationale no longer cites the issue.

## Rejected alternatives

- Raddle validating against the board body when the body is newer: the body is free markdown, not the record format, so criteria, risks, PATTERN changes and decision ids cannot be checked against it.
- Separate `lym loom decision edit <slug> D<n>` and `remove` verbs: whole-record replace covers both. Removing a decision also needs its acceptance bullets changed, which a per-id verb cannot do cleanly.
- Discussion-Burler-Fix folding added entries, with Go stripping them at approval: an entry added in the settling window after the last fixer round would be stripped unfolded and lost.
- A content marker in the record to reset the history check: a writer could forge it. Only a lym verb writes a commit trailer.
- A history reset keyed on the trailer commit touching the record's path: a sweep commit can land the bytes first, so the cut is keyed on the bytes' hash.
- A new `lym board record` verb: `upsert --record-file` lets the body and the record change in one board write, matching `--body-file`.
- A `revised` against `record_stored` time comparison for the promote warning: `revised` moves on the Sketch conversion, label edits and payload-set values, so it warned on every motif-made Sketch.
- A promote refusal for a stale record: a body change need not change the design.
- `--record-file` on an entry with no stored record: it would make an entry discussed with no Discussion.
- `--amends-plan` on `replace`: the amendment roll-back and its readers key on added-entry titles, which a replace does not write.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Should raddle follow the body or a replaced record? A replaced record (D1, D4), because raddle's checks need the record format.
- Q2 (auto-pick): Per-decision verbs, or a whole-record replace? A whole-record replace (D1), because folding changes several entries at once.
- Q3 (auto-pick): How is a replace made safe against a concurrent `add`? A required `--base-sha256`, checked under a per-run decision lock that `add`, `replace` and `store` share.
- Q4 (auto-pick): When is replace admitted? Wherever `add` is, except before Discussion approval on a recipe with that row.
- Q5 (auto-pick): How does the dropped-entry check stay valid after a replace? The trailer names the sha256 of the written bytes, and the history is cut at the newest version that matches it (D2).
- Q6 (auto-pick): Should a raddle run's design call reach the Sketch's board record? Yes, for every run whose entry carries a stored record (D3).
- Q7 (auto-pick): Which board surface? `upsert --record-file` (D4), refused while a run holds the entry or the slug has a live batten run, and refused for a run's task session.
- Q8 (auto-pick): How does promote detect a stale record? It warns when the body's sha256 differs from `record_body_sha256` (D5), and never refuses.
- Q9 (auto-pick): Who folds added entries? The parent, by `replace` on a run or by `--record-file` on a Sketch (D6). Fixers keep their hands off.
- Q10 (auto-pick): Does `replace` amend an approved plan? No. A call that needs an amendment goes through `add --amends-plan` first and is folded later.
