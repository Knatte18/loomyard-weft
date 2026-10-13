---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:43
    summary: D2's widened refusal reads a verify-failure record that outlives its halt, so a non-verify blocked Merge-In stop with a stale record is refused too, which falsifies the Bound's claim that every other blocked stop keeps its eligibility
    fix: In D2's Decision, require the record to belong to the halt being judged, not only to name a target - its row is the run's current producer (Merge-In or Publish), its head is the pair's HEAD and it was written during the halting attempt, the same match battenshed's retry makes before acting - and say the Publish check takes the same match. Add an acceptance criterion - a blocked Merge-In stop of another cause (a merge conflict or a dirty parent) while an earlier verify's failure record naming a target is still on disk stays eligible.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:43
    summary: D2's Bound does not say what a Merge-In verify halt loses when batten does not retry it - a standalone run with no batten, or a batten run whose one retry is spent or capped - where the arbiter could settle it today and the stop now goes straight to the parent
    fix: Append to D2's Bound one sentence - with no batten, or with batten's one retry spent or capped, a Merge-In verify halt whose record names a target is no longer settled by the arbiter and parks for the parent, as the Publish halt already does.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:61
    summary: D4 leaves open whether a symbol target such as a file-and-symbol glyph names its file for the file-level comparison, and gives no Bound for what the file-level comparison lets through or reports
    fix: In D4's Decision, state that a changed file with no symbols in the delta is exempt when it is the file part of any card target, a symbol target or a file-only target alike, taken through glyph's conversions. Add a Bound line - the list lets through a change outside any symbol (imports, file header, comments) in a file a card targets, and reports at file level every changed file the delta gives no symbols for, unsupported and extraction-error dispositions included, unless a target's file part names it. Add a matching acceptance criterion for an import-only change in a targeted file.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:64
    summary: D4 and its acceptance criterion name the PR narrative as a reader of the deviation list, but the landing Describe stencil, which writes the PR body, forbids describing glyph deviations; the narrative that reports deviations is the Master stencil's summary narrative
    fix: In D4 (line 64) and the acceptance criterion at line 118, replace the PR narrative with the Master stencil's summary narrative at its summary path, so the plan writer does not edit contracts/stencils/landing/landing-template-describe.md.
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:72
    summary: D5's landing-order rule (furthest along by recipe row) names no source for the row order the alignment agent compares, while lyx loom peers gives each peer only a state-and-producer status
    fix: In D5's Decision, name where the agent reads the row order - the stencil points at the hub's loom recipe file rather than restating the rows, per PATTERN-producer-pointer-rule - and state that the comparison uses the producer half of the peer's run status, so a peer whose producer is not a row of that recipe is the unknown order D5 already names.
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:78
    summary: D5's Bound names only the dropped text-only escalation; it does not state that a text-only overlap can hide a semantic interaction caught only by the second-landing run's Merge-In verify, nor that the PATTERN-entry escalation narrows from any shared entry to one changed in different ways
    fix: Extend D5's Bound - a text-only overlap that interacts in behaviour (one run changes a signature, the other adds a caller elsewhere in the package) is caught only by the verify at the second run's Merge-In, and two runs changing the same PATTERN entry the same way no longer escalate; both hold at parent-review and at pr-gate, where an approve lands the run with no orch turn.
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:28
    summary: D1's relaxed pause check is written for every consume, so an unmoved run whose pause condition was removed by lyx shed pause --clear during the agent's work is now admitted where today it is refused, and the Bound does not say so
    fix: Either scope the relaxed pause check to the verdict-in-effect admit (a condition the goto cleared does not refuse; an unmoved run keeps today's equality check), or add to D1's Bound that a pause condition cleared by hand since the spawn no longer refuses the re-launch. Name the choice in an acceptance criterion.
---

### [MEDIUM] D2's widened refusal catches non-verify Merge-In stops

**Location:** `_lyx/discussion/decision-record.md:43`

**Issue:** The arbiter decides the Publish refusal from `verifytree.ReadPublishFailure`'s `Target` alone (`internal/loomcli/arbiter.go`, `checkEligible`).
The record is written on a failed verify and removed only by a later passing verify (`internal/landingshed/verifygate.go`, `recordFailure` and `clearFailure`), so it stays on disk after the halt it describes.
Merge-In has many blocked stops that are not verify failures: a parked merge, a dirty or moved parent, a dirty tree (`internal/landingshed/doc.go`).
After a Merge-In verify failure, batten's retry moves the run to the target, and the run comes back to Merge-In with the record still present; if Merge-In then halts on a merge conflict, D2 as written refuses the arbiter for a stop that has nothing to do with verify.
The Bound's "every other blocked stop keeps its eligibility" is therefore false.
battenshed already holds the right match (`RecordMatchesHalt`: row, head, write time against the halting history entry).

### [LOW] D2's Bound omits what a Merge-In verify halt loses with no batten retry

**Location:** `_lyx/discussion/decision-record.md:43`

**Issue:** The Decision says "batten's retry, or the parent when no batten retries, takes the run on", and the Rationale says batten's retry covers the stop.
Batten retries once per run (`internal/battenshed/doc.go`), and a standalone run has no batten.
In those cases a Merge-In verify halt the arbiter settles today goes to the parent after this change.
That is the cost the rubric's attack-surface question asks the Bound to state; it is stated for neither case.

### [MEDIUM] D4's file-level comparison leaves the target-file match open and has no Bound

**Location:** `_lyx/discussion/decision-record.md:61`

**Issue:** `ScopeGuard` (`internal/planglyph/scope.go`) compares symbol IDs against the cards' target glyphs.
The delta's `Files` list gives every changed file with its disposition, so the file-level comparison can be built, but "unless a card target names that file" does not say whether a symbol target names its file.
If it does not, every import-only or comment-only change to a targeted Go file is listed, which is the noise D4 sets out to remove.
If it does, a change outside any symbol in a targeted file is let through unseen.
The plan writer must make that choice, and D4 carries no Bound saying what the list now skips or reports.

### [LOW] D4 names the PR narrative as a reader of the deviation list

**Location:** `_lyx/discussion/decision-record.md:64`

**Issue:** `contracts/stencils/landing/landing-template-describe.md` writes the PR body and says never to describe glyph deviations.
The narrative that reports deviations today is the Master's summary narrative (`contracts/stencils/webster/webster-template-master.md`, the `{{.summary_path}}` paragraph).
"The PR narrative" in D4 and in the acceptance criterion at line 118 points the plan writer at the wrong stencil, or at one whose rule D4 would contradict.

### [MEDIUM] D5's landing-order rule names no row order

**Location:** `_lyx/discussion/decision-record.md:72`

**Issue:** `lyx loom peers` prints each peer's board status, whose run-status form is `<state> · <producer>` (`internal/boardengine/runlock.go`).
"Furthest along by its recipe row" needs the recipe's row order, and neither D5 nor the alignment stencil it edits gives the agent one.
Without it the rule is not a decidable test: the agent would guess the order or escalate.
The tie and unknown cases are covered; the source of the order is not.

### [LOW] D5's Bound states only part of what the removed escalation lets through

**Location:** `_lyx/discussion/decision-record.md:78`

**Issue:** The current stencil escalates on any Go package or code file both runs change, and on any change to the same PATTERN entry.
D5 narrows both.
The Bound covers textual ordering through the parent lock and Merge-In.
It does not cover a behavioural interaction between text-only overlaps, which only the second run's Merge-In verify catches, or the narrowed PATTERN-entry rule.
At `pr-gate`, an approve lands the run with no orch turn, so the Bound should name what bounds a missed conflict there.

### [LOW] D1's relaxed pause check also admits a hand-cleared pause on an unmoved run

**Location:** `_lyx/discussion/decision-record.md:28`

**Issue:** Both consumes compare the pause conditions for equality today (`internal/loomcli/arbiterconsume.go`, `internal/loomcli/tacklerconsume.go`).
D1 replaces that with "refuses only a pause condition present now that the attempt did not record at spawn", stated for the pause check as a whole.
`lyx shed pause --clear` removes both conditions (`internal/shedverbs/pause.go`), so an unmoved run whose pause was cleared while the agent worked is admitted after this change.
That may be wanted, but it is a guard relaxation outside the verdict-in-effect case, and D1's Bound does not state it.

## Rejected

- D1 acceptance coverage (focus item 1): every admit condition has a criterion that exercises it, either directly or as the refused counterpart. A different row, a goto then a step, a resume with no goto, a cleared and a new pause condition, the hub pause and the tackler parity are each named. The `paused` condition is reached through the goto-then-stepped criterion, since a goto always leaves the run paused. Not a defect.
- D1's Bound on a parent's goto to the same row (focus item 1): it states the skip (stepping without the parent's resume) and its precedent. `internal/loomcli/drive.go`'s `recheck` does re-step after any trailing goto entry, so the precedent holds. Not a defect.
- D2's premise (focus item 2): `internal/battenshed/doc.go` and `publishretry.go` confirm batten's first-verify-failure retry acts at Merge-In and Publish with a goto and a resume. The Open risks line on a different mover is consistent with the Rationale. Only the Bound gaps in F1 and F2 remain.
- D1's tackler extension: the Go driver already rechecks and re-steps a moved run before and after a refused tackler consume, so the shared admit changes little there. The support log records the choice and its reason (Q3), and it is not a defect.
- Requirements carried only in Rationale or the support log (focus item 5): none found. Each Rationale and each support-log disposition is reflected in a Decision, Constraint or criterion.
- Relocation (focus item 6): the dated and incident clauses in D3's and D5's Rationale are each the decision's sole justification and one line long. Moving them would leave the Decision without a reason, so no relocation finding is raised.
