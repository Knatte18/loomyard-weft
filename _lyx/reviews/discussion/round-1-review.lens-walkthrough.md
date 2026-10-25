```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each Decision's mechanism end to end on one concrete case and checked the end state against the Decision's purpose.

**List — each Decision, the case I used, purpose met or not**

- D1: delete `internal/alignment` and move `DecisionNoteTitle` for its callers (`loomcli/decision.go`, `orchengine/prompt.go`). Met.
- D2: a run waiting on a parent-review round when the driver restarts on the new binary; the gate list holds only `discussion`, so it passes. Met.
- D3: a hub `loom.yaml` that still holds `skip_alignment`; reconcile drops it and the strict load then passes. Met.
- D4: an audit window spanning the landing; `reportengine` parses the archived records. Met.
- D5/D6: a `goal-scope` fork in a `review-discussion` round reads its parent-decision notes; a `board-fit` fork spends its 6 calls. **Not met** (W3).
- D7: a writer's choice filed as `decision`; the rubric rules it out. Met.
- D8, case a: a CONVERGED round holds a `decision` finding; it escalates; a re-call returns the same Awaiting; `continue` runs round n+1. Met.
- D8, case b: the parent records `accept`, it settles, then the segment is re-entered (a commit-seam failure followed by a resume). **Not met** (W1).
- D8, case c: a CONVERGED round holds a `decision` finding and a BLOCKING `design` finding; the parent accepts. **Not met** (W2).
- D8, case d: the parent answers with `lyx loom decision add`, then accepts. Partly met (W5).
- D9: the writer leaves an outdated-entry question in Open risks, the perch raises it, and `continue` drops the line. Met.
- D10, case a: darn writer, outdated entry, a parent answer by note; the record is committed, then `lyx loom decision add`. Met.
- D10, case b: the run has no parent. Partly met (W6).
- D11: a run parked on `Merge-In` after spending its fix attempts, resumed after `RenameProducers`. **Not met** (W4).

**Findings**

- **W1 — MEDIUM, design — D8, "never a settled generation at Call entry".**
  - The rule says an escalated round is never a settled generation, with no exception once its accept is settled.
  - Case: the accept settles through `acceptRound` (`settleCirclingAccept`), the escalation file stays on disk, and the segment is re-entered later.
  - Today `settledGeneration` (bouncer.go:833–843) counts a settled accept as settled and archives the round (clear) or harvests it.
  - Under D8 the re-entry takes the recorded-decision path instead. `settleUnconverged`'s `case settled` then degrades (bouncer.go:1199–1200), and its way forward, `lyx loom start` "which re-enters the segment and archives the settled round", no longer archives anything. The run loops.
  - Fix: limit the rule to a `decision` escalation whose decision is absent or unsettled.
- **W2 — MEDIUM, design — D8, "What it can skip or let through: nothing".**
  - D8's check runs before `judgmentRefusal`, which refuses a CONVERGED verdict over a BLOCKING review finding or an open BLOCKING ledger entry (bouncer.go:1025–1043).
  - Case: a round holds a `decision` finding and a BLOCKING `design` finding, and the judge rules CONVERGED with the ledger entry closed.
  - D8 escalates, and the brief lists only the `decision` findings. The parent's `accept` goes to `acceptRound` (bouncer.go:1234–1253), which passes the round.
  - A refusal that would have forced a re-judge is skipped. The BLOCKING finding is neither shown to the parent nor carried over, because the closed ledger entry is not an open finding.
  - Fix: name this bound in D8, and either list every BLOCKING finding in the brief or apply the refusal before an accept settles.
- **W3 — MEDIUM, design — D6, the reads of both lenses.**
  - `goal-scope` reads "every `Decision for <slug>:` note", but none of its reads produces the note list: it has no `lyx board list`.
  - Each note body costs one more `lyx board get`, with no limit, under the fork's 6-call cap (`burler-cluster-fan.md`).
  - "Both lenses" lets a decision note override the entry, yet `board-fit`'s reads (list, peers, up to four gets) are exactly 6 and include no note read.
  - So `board-fit` cannot see a note that answers its question, and raises a `decision` the note already settled. That stops the run for nothing, which D7's exclusion was meant to prevent.
- **W4 — MEDIUM, design — D11, row-named scratch records.**
  - `FixerAttemptsPath` is `<row>-fixer-attempts.json` (fixerattempts.go:30–31). The record exists "so a resume never grants a fresh cap" (fixerattempts.go:1), and it also holds `AlignFrom`.
  - The fixer report is `<row>-fixer-report.md` (fixerrun.go:91).
  - D11 names a fallback for the report only.
  - Case: a run halted spent on `Merge-In` is renamed and resumes on `Parent-Align`. `ReadFixerAttempts` finds no file and returns a zero record (fixerattempts.go:35), so the entry gets a fresh `parent_align_fix_attempts` budget and loses `AlignFrom`.
  - A fixer still live at upgrade is not matched either, because the shuttle's Attach matches on output files and its report path now carries the new name, so a second fixer can spawn beside it.
- **W5 — LOW, consistency — D8 accept against D7.**
  - Case: the parent records the answer with `lyx loom decision add`, then accepts.
  - The carry-over still writes every `decision` finding into `## Open risks` "whatever the judge's ledger says", so Plan-Write gets an Open risks question beside the `Added after Discussion` entry that answers it.
  - D7 calls such an answered line a `consistency` defect, and no Discussion round follows to drop it.
- **W6 — LOW, design — D10, "With no parent, it stops and reports".**
  - The darn writer's one output is the change description. Ending its turn without it is held by shuttle until `writer_timeout_min` (template.yaml:2, 180 minutes).
  - With no parent there is no hold notice (shuttleengine/doc.go:89–91), so the "report" reaches nobody until the timeout, and nothing says how the stop surfaces.

Tool calls:
1. grep of `landingshed/fixerrun.go` and `mergein.go` for the fixer report path and output files: my context lacked how the row-named report is formed.
2. sed of `fixerrun.go:80-140`: my context lacked the fixer spawn, its attempts path and its table.
3. grep of `FixerAttemptsPath`, the `fixerattempts.go` header and its callers: my context lacked how the attempts record is named per row and why it exists.
