```yaml
lens: walkthrough
outcome: reported
```

**Walkthrough lens — decisions walked**

- D1 — case: a copy written into the kept directory, then a status commit, then teardown. Purpose met: `statusCommitPathspec` stages the directory when it holds a file, teardown's `commitPendingRecords` stages the structural `_lym` whole before `tagWeftTipForArchive`, and the landing gate's dirty check reads only the code worktree (`landingdeps.go:450`).
- D2 — case: a send-back, its re-run after a failed commit, then a fix-here re-run over the same file. Purpose partly met: the send-back re-run returns at `rec.SendBack != ""` before any write (no duplicate), but a fix-here re-run of an already-counted file, or a retry after a failed commit, takes the next free number and copies the same findings again.
- D3 — case: an ask recorded by the seat, answered by the orch's `lym loom parent-review`, then settled. Purpose met: every write goes through `state.UpdateJSON` closures in `parentreview.go`, and the orch process resolves the target's anchor (`loomcli/parentreview.go:400`).
- D4 — case: Finalize's fixer with cap 3, turn 1 reports fixed but the gate fails, turn 2 passes. Purpose missed (F1).
- D5 — case: a landing through a GitHub pull request, then Finalize resumed after other runs land on the parent. Purpose missed (F3).
- D6 — case: `--slug` over a slug with one landed tag and one cancelled-run tag, then `--extract`. Purpose partly missed (F5).
- D7 — case: trace the cited carve-out in `cleanupremotewarp.go`. Claim misdescribes code (F7).

**Findings**

1. **BLOCKING · design** · `decision-record.md` D4 / its acceptance criterion "two attempts in one row … leaves two tracked reports"
   One fixer *session* spans every turn: `fixerRun.run` gives one `f.seats.Run` a gate with `Attempts: turnsLeft`, and a failed gate re-prompts in the same session after `chargeTurn`, each turn overwriting the one `reportPath`. A copy "after each landing fixer session ends" therefore keeps only the last turn's report; the criterion's case (first attempt failing, second passing, in one row) leaves one report, not two.
   Evidence: `internal/landingshed/fixerrun.go:105` (one `reportPath` per row), `:139-143` (one session, `Attempts: turnsLeft`), `:327-334` (charge and re-prompt in-session).

2. **MEDIUM · design** · `decision-record.md` D4 "named by row and by the attempt number from the attempt record"
   The attempt number is not unique per row: `ReadFixerAttempts` returns a zero record when the entry changes, so `Spent` restarts on a new row entry (e.g. Parent-Align re-entered after a send-back), and an escalate turn decrements `Spent`, so the next session reuses that number. A later copy overwrites an earlier one.
   Evidence: `internal/landingshed/fixerattempts.go:79-81`, `fixerrun.go:296-297`.

3. **BLOCKING · design** · `decision-record.md` D5 "A resumed Finalize rewrites the row with the same content, since every field comes from durable records"
   The landing commit is in no tracked record: `recordLanding` reads `parentHandle.HeadSHA()` (the parent's current head) and `RecordLanding` stores it only in the prime's scratch via `shedrun.RecordLandedCommit`. A Finalize resumed after other runs landed on the parent reads a different commit and committer time, so the "byte-identical row" criterion fails.
   Evidence: `internal/landingshed/finalize.go:738-753`, `internal/loomcli/landingdeps.go:249-256`.

4. **LOW · consistency** · `decision-record.md` D5 row vs D6 extract
   The row records "the worktree-relative path of each kept record", while `--extract` writes "`_lym/`-relative paths"; the two bases differ, so a reader resolving a row path against `&lt;dir&gt;` misses the file. The record names no single base.
   Evidence: decision record D5 para 2 vs D6 `--extract` bullet.

5. **MEDIUM · design** · `decision-record.md` D6 list and `--slug` for a non-landed tag
   A cancelled run's tag exists (`battencli/cancel.go:822` reports `ArchiveTags`) with no landing; D6's "derived" line for it would show a landing time/commit the run never had. The record does not say how a non-landed tag is shown or marked, though `reportengine` already separates such tags as `PartialRun`.
   Evidence: `internal/battencli/cancel.go:152-153,822`, `internal/reportengine/runs.go:114-121,163-166`.

6. **MEDIUM · design** · `decision-record.md` D2 "each takes the next free number … so no copy overwrites another"
   A fix-here re-run over an already-counted file (`counted` true) still reaches `answer`, and a retry after a failed `deps.commit` reruns the verb; each would copy the same findings again under a new number. D2 bounds overwrite but not duplication; the copy needs keying to the counted hash or skipping for a counted set.
   Evidence: `internal/loomcli/prfindings.go:204-209`, `:222-227`.

7. **LOW · consistency** · `decision-record.md` D7 "apart from the existing cleanup in `internal/fabricengine/cleanupremotewarp.go`"
   That code deletes nothing: `PlainArchiveTagsOnOrigin` only lists plain tags and returns a way-forward command for the operator. D7's carve-out describes a deletion that does not exist.
   Evidence: `internal/fabricengine/cleanupremotewarp.go:336-379`.

Tool calls:
1. grep `verifyGate.dirtyPaths` / landing `DirtyPaths` wiring — to check whether a kept-directory copy shows as dirty to the fixer gate; context lacked it.
