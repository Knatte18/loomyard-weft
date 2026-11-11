```yaml
lens: walkthrough
outcome: reported
```

**Walkthrough lens: one concrete case per decision, run step by step on the data each step really receives**

- **D1** (tapestry run at Plan-Write; the parent folds two added entries and passes the correct base hash): purpose met. The working copy is checked against HEAD, written in place, validated against record and support log, then committed with the trailer through `CommitAnchoredPathsWithTrailers`; the empty commit lands when nothing is staged. The board copy and the notice follow. A raddle run at Raddle-Write follows the same path through `checkDarnRecord`.
- **D1, git step** (`git commit --allow-empty -m … -- <record>` while another path is staged): met. On git 2.53.0 the handler's experiment landed an empty commit, and the other path stayed staged.
- **D2** (replace drops entry E, and a sweep commit C1 had already landed the bytes; the empty trailer commit C2 touches no path): met. `PathRevisions` lists C1 and not C2. The newest version hashing to the trailer's sha256 is C1, so the cut drops the older versions that held E. An entry F added later and then dropped is still flagged.
- **D3, case 1** (raddle run of a promoted Sketch; `add` at Raddle-Write): met. The raddle recipe has no approval row, the entry carries a stored record, and `StoreDecisionRecord` copies it past the run lock.
- **D3, case 2** (motif or tapestry run re-entered into its Discussion segment after an earlier approval; the parent runs `add`): broken, see F1.
- **D3, case 3** (non-perch family filing done; `add` on the run): met. `raddleRunFiling` reports done, and the refusal keeps the project's record.
- **D4, case 1** (Sketch note after the motif pair was torn down; operator runs `upsert --record-file`, then promote): met. Record-Write finds no record on the fresh weft and seeds the replaced stored record, which raddle then reads.
- **D4, case 2** (same, but the raddle pair adopts the motif's still-live weft): broken, see F2.
- **D5** (motif approval stamps body B; Sketch conversion; promote): met. The body is unchanged, so no warning. After a body edit to B2 with no record write, it warns.
- **D6** (parent directive; parent orch folds a Sketch with `--record-file`; a spawned role gets the replace notice): met. `isRunAgentSession` lets the orch strand through, and the directive names the new notice opening.

**Findings**

F1
- Severity: MEDIUM. Class: design.
- Location: D3, first paragraph ("or when the run's board entry carries a stored record"), and its `store` admission.
- Summary: the new condition is not limited to recipes that have no Discussion approval row.
- Case:
  1. A motif or tapestry run passes approval, which stores the record and sets `discussed`.
  2. The run re-enters its Discussion-Bouncer segment, a re-entry D1's own Bound describes.
  3. `PastDiscussionApproval` now reports false, but the entry still carries a stored record.
  4. `add` copies the record onto the board in the middle of review, and `store` is admitted there too.
- This undoes the existing rule "A call made while `Discussion-Bouncer` is settling is not copied to the board". If the run is then cancelled, the Sketch keeps a record no review approved.
- Fix: apply the stored-record case only to a recipe with no approval row, and keep the past-approval gate for every recipe that has one.
- Evidence: `internal/loomcli/decision.go:5-6`, `:534-536`; `internal/loomrecipe/discussionapproval.go:44-75` (a row inside the approval segment is not past).

F2
- Severity: MEDIUM. Class: design.
- Location: D4's weft bullet, and Open risks bullet 4.
- Summary: with an adopted live motif weft, raddle never reads the `--record-file` record, and D3 then overwrites it on the board.
- Case:
  1. The motif teardown's remote step failed, so the raddle pair adopts the motif's weft branch.
  2. That branch already holds the motif's committed `_lym/discussion/decision-record.md`.
  3. Record-Write writes a record only when the file is missing ("A file that exists is never overwritten"). So the run keeps the motif's stale record, and the operator's folded board record never reaches it.
  4. Raddle splits by the stale design, which is the #116 defect.
  5. The first `add` or `replace` on that run copies the stale committed record back onto the board through D3, erasing the operator's `--record-file` write.
- Open risks bullet 4 misstates this end state. It says the dropped-entry check flags the folded entries and a self-replace clears it; in fact the record still holds the unfolded entries, so nothing is flagged and nothing tells anyone the record is stale.
- Fix: decide the case. Either Record-Write re-seeds from the board's stored record when it differs and the run has not yet written, or `--record-file` is refused while the slug's weft branch is still live.
- Evidence: `internal/loomshed/recordwrite.go:120-124`, `:136`, `:153-158`; `internal/fabricengine/add.go:150-157`; `internal/loomcli/decision.go:534-544`.

Tool calls:
1. `internal/loomrecipe/discussionapproval.go:10-75`: my context had only `PastDiscussionApproval`'s signature, and F1 needed to know how a row inside the approval segment is judged.
