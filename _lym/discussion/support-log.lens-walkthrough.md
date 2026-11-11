```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end against the code as it stands.

**Findings**

1. **BLOCKING** (D5): every Sketch a motif run makes would warn at promote, so the warning means nothing.
   - The case: a motif run reaches Discussion approval. `ApproveDiscussion` stores the record, so `record_stored` is T1.
   - `sketchFinish` then calls `boardengine.Board.MakeSketch` (`internal/boardengine/sketch.go`). That turns the entry into a note with the `sketch` label and clears `recipe`.
   - `scopeChanged` (`internal/boardengine/runlock.go`) counts changes to Kind, Labels and Recipe. So the stamp moves `revised` to T2, later than T1, in the same run.
   - At `lym board promote`, `revised` is later than `record_stored` on every motif-made Sketch even when nobody touched the body. The warning stops telling a stale record from a fresh one, which defeats D5's purpose.
   - D5 needs either a body-only revision time or a comparison that ignores the Sketch conversion.

2. **MEDIUM** (D4): the check D4 says it runs cannot run on a board record, because the board keeps no support log.
   - D4 says the file "must pass the section check … through `internal/discussionparser`". The only section check is `discussionparser.Validate(decisionRecordPath, supportLogPath)`.
   - `Validate` checks the support log first, and a missing file "yields one Finding … and ends the check" (`internal/discussionparser/validate.go`). Every `--record-file` would therefore fail with `discussion-file-missing`.
   - D4 needs to name a section check that reads the record alone, and the plan has to add it.

3. **LOW** (D5): the record leaves open which `revised` value promote compares.
   - Promote turns the note into a task with recipe raddle. That is a scope change, so `revised` moves inside the promote's own write.
   - D5 does not say whether the comparison uses the entry as loaded before the write. If it uses the value after the write, it warns every time.

4. **LOW** (D4, D5): the "times are equal, no warning" criterion depends on one clock reading.
   - An upsert carrying both a body and `--record-file` must stamp `revised` and `record_stored` from the same `now`.
   - `stampDates` and a separate record stamp could fall either side of a second boundary. Then `revised` reads later and the warning fires on a write that updated both. D4 and D5 should say they share one timestamp.

**Each decision, the case I used, and whether its purpose is met**

- **D1**, a parent replacing the record on a tapestry run at Plan-Write: met.
  - The parent hashes the working copy. The precondition makes it equal the committed copy, so the base matches.
  - `Validate` reads files from disk, so the file has to be written into place and restored on findings, as `AppendDecisions` does. That is plan-level.
  - The carry-over check, the commit, D3's copy and the notice follow in order.
- **D1**, a held raddle run, the issue #116 case: met.
  - Raddle has no Discussion approval row, so replace is admitted. `checkDarnRecord` passes on the committed record.
  - The raddle gate re-validates against the new file after `lym loom goto --to Raddle-Write` and resume.
- **D2**, a replace that drops an added entry, followed by a gate run: met.
  - `statusCommitPathspec` (`internal/loomcli/wiring.go`) does not stage the discussion directory. So a status commit cannot carry the replaced record without the trailer.
  - `discussionCommitPathspec` does stage it, but only the Discussion rows commit through it, and D1 refuses before approval.
- **D3**, `add` on a raddle run of a promoted Sketch: met.
  - `checkDiscussedPromotion` keeps the `discussed` flag on the promoted task. So "the entry carries a stored record" holds, and `StoreDecisionRecord` refreshes the record.
- **D4**, the operator upserting a Sketch's body and `--record-file` together: not met (finding 2).
- **D5**, motif, then Sketch, then promote with no body edit: not met (findings 1 and 3).
- **D6**: prose only, checked by review. I traced no mechanism.

**Tool calls**

1. `internal/discussionparser/validate.go` head: whether `Validate` needs a support log. My context held only its signature.
2. `scopeChanged` and `MakeSketch`: which changes stamp `revised`, and what the Sketch conversion changes. My context had only the subagent's summary.
3. A grep for "status commit" in `decision.go` and `wiring.go`: whether status commits stage the record.
4. `statusCommitPathspec` and `discussionCommitPathspec` in `wiring.go`: the exact pathspecs.
