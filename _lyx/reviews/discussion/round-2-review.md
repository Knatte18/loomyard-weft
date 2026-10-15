---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:87
    summary: D8 hands the re-measurement to the orch at landing, but nothing delivers D8 to the orch and this hub lands an agent-approved run with no orch turn, so the board entry's re-measure ask is silently lost
    fix: In D8, replace the sentence starting "The orch, at landing, files a board note" with one stating that the discussion writer asks its parent (ly:orch) during this run, by message, to file the board note now through `lyx board upsert` with kind note, naming this task's slug and carrying the same measurement content (forkaudit over post-deploy Webster runs, per-fork total reads and failed Edit calls against the 2026-10-12, -13 and -15 audits, the Master's own reads as the D1 compliance check). Change the Rationale's last sentence to say the orch files proposals as board notes at once and the note survives the entry's close. In support-log.md, update ledger item Q9 to the new carrier and record that the request was sent.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:56
    summary: D3's replacement statement tells every fork the plan and rules it holds stay current, which the first Open risk says is false when a card of the run edits PATTERN.md, a pattern/ file or CLAUDE.md
    fix: In D3, change "the plan and rules you hold stay current" to "the plan and rules you hold stay current, except a rules file a card of this run targets, which you read fresh like code".
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:3
    summary: The Goal says the Master reads every pattern/ background file PATTERN.md points at, while D1 restricts it to those touching what the forks will do
    fix: In the Goal's first sentence, change "and the `pattern/` background files it points at" to "and the `pattern/` background files it points at that touch what the forks will do".
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:58
    summary: D3 says the rest of each fragment, the Edit-only rule included, is unchanged, but the plain fragment has no Edit-only rule and D3 itself changes its read sentence
    fix: Replace D3's sentence "The rest of each fragment (what to read before the first edit, the Edit-only rule after an edit) is unchanged." with "Every other sentence of each fragment is unchanged, apart from the plain fragment's re-read to read."
---

### [BLOCKING] D8's carrier is never delivered to the actor it names

**Location:** `_lyx/discussion/decision-record.md:87`

**Issue:** D8 makes the orch, at landing, file the board note that carries the board entry's "re-measure with `tools/forkaudit` afterwards" ask.
Nothing hands D8 to the orch.
The orch role stencil's `## After landing` (`contracts/stencils/orch/orch-template-role.md:129`) says "A landing needs nothing more from you than these steps", and those steps never read the decision record.
The orch reads the decision record only through its PR-Gate reviewer fork, and that fork reads only the carried-over `Merge-Review` open risks (`orch-template-role.md:116`).
That path runs only when the alignment agent escalates: the hub's `_board/_lyx/config/loom.yaml` sets `alignment_pr_approve: true`, and "A run the agent approved lands with no orch turn" (`orch-template-role.md:112`).
The board entry, which holds the "afterwards" ask, is marked done at Finalize and pruned by batten's Land row.
D8's own rationale already rules out a card and the task summary as carriers, which matches `webster-template-master.md:243`, where the Master writes the summary only from the reports and digests it read.
So, as written, the re-measurement is dropped with nothing to signal the loss.
The discussion writer cannot write the board itself, since its stencil leaves everything but its two files read-only.
Its parent can: the orch's board policy is "A proposal goes on the board as a note at once" (`orch-template-role.md:144`), so a request sent during this run gives D8 a carrier that exists.

### [LOW] D3 claims rules stay current where the Open risks say they may not

**Location:** `_lyx/discussion/decision-record.md:56`

**Issue:** The replacement text that every live FRESH-READ fragment renders tells the fork that "the plan and rules you hold stay current".
The first Open risk says a plan whose cards edit `PATTERN.md`, a `pattern/` file or `CLAUDE.md` leaves the Master's copy stale for later forks.
The stencil text then states, unconditionally, something the record knows to be false in that case.
The bound the Open risk names (the editing fork reads its target fresh) holds only if the fragment's wording does not tell that fork its rules copy is current.

### [NIT] Goal states a wider pattern/ read set than D1

**Location:** `_lyx/discussion/decision-record.md:3`

**Issue:** The Goal says the Master reads "the `pattern/` background files it points at", which reads as every linked file.
D1 and the orchestrator directive restrict the read to files that touch what the forks will do.
The plan writer reads both sections, and the Goal should not name a broader set than the Decision.

### [NIT] D3's "rest unchanged" sentence does not fit the plain fragment

**Location:** `_lyx/discussion/decision-record.md:58`

**Issue:** `webster-fragment-fresh-read-plain.md` has no Edit-only rule, and D3's previous sentence changes its read sentence (re-read becomes read).
So the sentence claiming the rest of each fragment is unchanged is wrong for the plain fragment on both counts.

## Rejected

- D1 and D4 pattern/ read set: each role has one definition. The Master renders the orchestrator directive and reads what touches what the forks will do, filtered from the Card Index and `## Shared Decisions` (D1). The recovery strand renders the implementer directive and reads what touches what it is about to change (D4). In-session forks render no pattern directive and inherit the Master's set. No defect beyond F3's Goal wording.
- D3 replaced spans: the three named spans match lines 6, 7 and 8–9 of `webster-fragment-fresh-read-pack.md` and `-pack-card.md`, and lines 6, 7 and 9–10 of `-plain.md`, verbatim. The kept text (pack-file pointer, per-card step 1, Edit-only rule, plain read list) does not contradict D1. For the cold recovery strand, "no code reaches you from Master" is true, and with re-read becoming read the plain fragment no longer presumes an earlier read. The one wording defect is F2.
- Acceptance criteria against a conditional-free master template: each one can be met by static wording and checked by a render-and-assert row. The empty-`pattern_directive` criterion holds because D1 words the pattern/ read as relative to "the constraints section above, when present", which a template test can check as a phrase rendered with the directive empty. The recovery criterion "holds for a session that inherited nothing" is checked through D3's fixed wording (read, not re-read). No defect.
