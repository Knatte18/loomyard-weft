```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that meet each Decision's state.

**Paths traced, per decision**
- D1: `DecisionNoteTitle` callers (`loomcli/decision.go` `decisionWayNoRecord`, `orchengine/prompt.go` marker). Survives the move.
- D2: Discussion-Write gate list, `lyx loom status` note, batten `ReviewWait`, the writer stencil's delivery and keep-alive carve-outs. All are named for removal. Survives.
- D3: hub and worktree reconcile, plus read-time migration for profiles and `run_settings`. Survives.
- D4: reportengine reads of archived rounds through its own parse. Survives.
- D5: the dormant `discussion` fan also holds `goal-scope`, and a hub `lenses.yaml` that tuned either lens keeps its own text. **Breaks**, see F4.
- D6: the 6-call cap in perch forks and in twill forks (`twill-directive.md:35`). `board-fit` survives. `goal-scope`'s "every `Decision for <slug>:` note" read count is not bounded; the bound-breaker lens owns that.
- D7: the rubric's class definition and the fix row's decision withholding. Survives.
- D8: `settledGeneration` after an accept has settled, and the settled-entry new generation over a changed subject (amendment). **Breaks**, see F2.
- D8: `standsAsRecorded`, then `settle`, then `acceptRound` with `judgmentRefusal` skipped. **Breaks**, see F3.
- D8: `settleUndecided`'s `!escalated` auto-accept check survives. `unearnedCircling` with a decision file present survives. The causes are validated in `escalation.go` `valid` and `circling.go` `readCirclingFile`, which the Notes name. `arbiterstop.Eligibility` already refuses any cause other than circling and hard-cap. Survives.
- D10: `decisionWayNoRecord` already tells the parent to message Darn-Write or write a `Decision for <slug>:` note. Darn Step 3's record condition and the shuttle hold also apply. Survives.
- D11: `FixerAttemptsPath(scratchDir, row)` is the per-row attempt record that Merge-In reads and removes. **Breaks**, see F1.
- D11: the landing failure record's `Kind` is the gate key; readers use only `LogPath`. Verifytree command entries are keyed by command, so the first verify after the rename runs once more. `BaseCommand` is `code-burler-fix`, which the rename does not touch. Reportengine's `rowFamily(NameMergeIn)` is covered by D11. Survives.

**Findings**

F1 — MEDIUM, design. Location: D11 (`decision-record.md:157-158`).
Summary: carrying a mid-entry run across loses its Merge-In fixer attempt record.
- The record is named after the row: `landingshed/fixerattempts.go:30-31` builds `<row>-fixer-attempts.json`, written and read at `fixerrun.go:92`, removed at `mergein.go:208`.
- A run renamed onto `Parent-Align` by `RenameProducers` reads `Parent-Align-fixer-attempts.json`, which does not exist.
- `ReadFixerAttempts` returns a zero record for a missing file (`fixerattempts.go:34-35`). The entry then gets:
  - `Spent` 0, a fresh `parent_align_fix_attempts` budget.
  - No `Baseline`, so the earlier fixer commits are never judged against the caps (`mergein.go` `judgeFixerChange`: "An entry whose fixer never ran has no baseline and no change").
  - No `AlignFrom`, the parent tip the alignment reads from.
  - No `Halt`.
- D11 adds a fallback only for "the row-named fixer report the Code-Perch reads", so the over-cap guard is bypassed with no bound stated.
- Fix: read the old row-named attempt record when the new one is absent, as D11 already does for the report.

F2 — MEDIUM, design. Location: D8, second bullet (`decision-record.md:108`).
Summary: "never a settled generation at Call entry" also catches a round whose accept has already settled.
- Today `settledGeneration` returns true for a settled accept (`shedadapters/bouncer.go:833-843`). That is what lets a later entry re-settle, or start a new generation over a changed subject (the amendment path), or clear.
- D8's rule covers every escalated round, so a re-entry after the accept settled takes the recorded-decision path instead.
- There it hits `settleUnconverged`'s `case settled` degrade (`bouncer.go:1199-1200`). Its way forward is `lyx loom start`, "which re-enters the segment and archives the settled round". But the archive happens only in the settled-entry clear, which D8 excludes, so the re-entry loops.
- An `Added after Discussion` amendment over such a record would also never start a new generation.
- Fix: limit the rule to rounds whose decision is absent or not yet settled.

F3 — MEDIUM, design. Location: D8, first bullet and "What it can skip or let through: nothing" (`decision-record.md:107,116`).
Summary: an accept on a `decision` escalation passes a round `judgmentRefusal` would have re-judged.
- D8 runs before the refusal. On the re-call, `standsAsRecorded` is true and the round goes straight to `settle` (`bouncer.go:494-497`, `1012-1016`). The accept then reaches `acceptRound` without the refusal ever running.
- So a CONVERGED round whose review also holds a BLOCKING `design` finding, or whose ledger drops a key, passes on the parent's accept.
- The brief lists only the `decision` findings (D8, third bullet), so the parent is not told about the BLOCKING finding. If the ledger marks it closed, no carry-over records it either.
- The stated bound "nothing" is false.
- Fix: list every BLOCKING and refused-judgment cause in the brief, or apply the refusal after the decision is settled.

F4 — LOW, consistency. Location: D5 (`decision-record.md:64-65`).
Summary: two fans and one hub layer that run these lens texts are not addressed.
- The rewritten `goal-scope` also runs in the dormant `discussion` fan (`internal/lenses/template.yaml:346-351`), whose other members are emphasis lenses. D5 names only `review-discussion` and `twill-discussion`.
- A hub `lenses.yaml` that tuned `goal-scope` or `board-fit` keeps its own text, because a name the file defines wins (`template.yaml:4-9`). Under D5 that hub's `review-discussion` would run the old emphasis text, with unbounded reads, inside a production fan.
- D5 states neither outcome.

**Tool calls**
1. `internal/lenses/template.yaml` fans grep, plus the twill directive's tool-call cap: to find every fan that holds the two lenses, and the twill fork budget.
2. `internal/lenses/template.yaml` fan lists: to see the full membership of `discussion`, `twill-discussion` and `review-discussion`.
3. `internal/lenses/template.yaml` header and the two lens texts: for the resolution rule and the current emphasis wording.
4. Grep of `landingshed`/`loomcli`/`loomshed` for verify base, mode and record paths: to find row-keyed state under the rename.
5. `internal/landingshed/mergein.go` and `fixerattempts.go` grep: to find where the attempt record is stored.
6. `FixerAttemptsPath` callers and the record's fields: to confirm the record is named after the row and what it carries.
