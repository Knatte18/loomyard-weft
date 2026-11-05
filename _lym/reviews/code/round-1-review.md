---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/sheddriver/drive.go:553
    origin: lens:code-correctness
    summary: noticeArbiterSettled tells the orch the run resumed and nothing needs answering before the arbiter's resume is consumed, so a refused consume leaves a false notice followed by a hand-back
    fix: >-
      In `arbitrate` in `internal/sheddriver/drive.go`, stop calling `noticeArbiterSettled` at the point where `stop.Restep` is set.
      Send the outcome line only after the re-step has consumed the arbiter attempt,
      for example by recording the pending notice on `goDriver` next to `pendingAttempt` and sending it once the consume succeeded.
      When the consume is refused and the run is handed back, send no outcome line.
      Update the `noticeArbiterSettled` comment and the `sheddriver/doc.go` sentence on the outcome line to say when it is sent.
  - id: F2
    severity: MEDIUM
    class: design
    location: contracts/stencils/loom/loom-template-arbiter.md:85
    origin: lens:code-correctness
    summary: The hard-cap and circling-at-cap rules tell the arbiter to record a decision and accept, but an accept ends the review segment so no run agent follows to carry the decision out
    fix: >-
      In the `hard-cap` rule and in the `circling` rule's at-the-cap branch of `contracts/stencils/loom/loom-template-arbiter.md`,
      drop the wording that the recorded decision is carried out by the run's own agents after an accept.
      Say that at or past the hard cap the arbiter accepts only when the finding is wrong or the fix in place already answers it,
      and records a decision there only to leave the ruling in the record;
      a fix that still has to be made goes to the parent as a hand-on question.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: contracts/stencils/bouncer/bouncer-fragment-decision.md:7
    origin: lens:code-affected-set
    summary: Prose still says a decision finding is a question only the run's parent can settle and tells the reader to put it to the operator, which contradicts the new arbiter-first routing of the decision cause
    fix: >-
      In `contracts/stencils/bouncer/bouncer-fragment-decision.md` reword the header comment (line 1) and the sentence at lines 7-8
      so a decision finding is a question the review cannot settle itself, settled by the arbiter where the board entry and the decision record answer it and put to the operator only when they leave it open.
      Make the same reword in `internal/shedadapters/doc.go:108` (the Decision escalation paragraph),
      in the `circling` group text in `internal/loomcli/circling.go:300`,
      and in `contracts/specs/shed-recipe-spec.md:61`, which says the escalation goes to the run's parent.
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/sheddriver/drive_test.go:452
    origin: lens:code-tests
    summary: Only the success branch of noticeArbiterSettled is covered, so the no-orch-recorded, OrchRecorded-error and failed-send paths, the awaiting-only guard and the notice text fail no test when broken
    fix: >-
      In `TestGoDriver_Settle` in `internal/sheddriver/drive_test.go` add rows for an arbiter resume at an awaiting stop with `orchRecorded` false (wantNotifies 0, run still finishes),
      with a `Notify` that returns an error (run still finishes),
      and for an arbiter resume at a blocked stop with `orchRecorded` true (wantNotifies 0).
      Add a `wantNotice` field to the row table that records the line `Notify` receives and assert it on the success row:
      it names the run, the producer and the verdict path and ends with `Nothing to answer.`
      Add the stubs these rows need to the harness (an `OrchRecorded` error and a `Notify` error field) rather than a second test function.
  - id: F5
    severity: LOW
    class: design
    location: contracts/stencils/loom/loom-template-arbiter.md:92
    origin: lens:code-correctness
    summary: The decision rule never tells the arbiter to read the brief's Refusal section before it accepts, so a held accept re-escalates and can end as a parent-only decision halt after the parent was told nothing was left to answer
    fix: >-
      In the `decision` rule of `contracts/stencils/loom/loom-template-arbiter.md` add one sentence:
      before choosing accept, read the brief's Refusal section;
      when a refusal is named, a held accept meets the same refusal again, so record the answers and continue below the hard cap,
      and hand on at or past the hard cap.
  - id: F6
    severity: LOW
    class: consistency
    location: contracts/stencils/loom/loom-template-arbiter.md:60
    origin: lens:code-affected-set
    summary: The stencil says its own rules win over the escalation brief's steps without naming which brief rules it replaces, while the brief it points the arbiter at still addresses a fork of the parent with rules the new text contradicts
    fix: >-
      In `contracts/stencils/loom/loom-template-arbiter.md` replace the sentence at line 60 with a list of the brief rules the arbiter does not follow:
      the brief's operator-decision-only rule for settling, its rule never to count a `--by arbiter` entry, its report-back-to-the-parent ending,
      and its rule against accepting over an open design-wrong finding;
      and state that every other line of the brief, the materials list and the refusal text included, still binds.
---

### [MEDIUM] The outcome line is sent before the arbiter's resume is consumed

**Location:** `internal/sheddriver/drive.go:553`
**Issue:** `arbitrate` calls `noticeArbiterSettled` as soon as the verdict is a resume, but the resume is applied only in the next step through `pendingAttempt` and `ConsumeAttempt`, which refuses an already-applied attempt, a changed stop key and changed pause conditions (`internal/shedarbiter/consume.go:50-59`).
On a refusal the run is handed back, so the orch has been told the run resumed and that there is nothing to answer, and then receives a hand-back notice.
The entry's "the orch hears only the outcome" is not met on that path, and the tackler path consumes before it settles (`drive.go:482-495`).

### [MEDIUM] Record-and-accept at the hard cap leaves the decision with nobody to carry it out

**Location:** `contracts/stencils/loom/loom-template-arbiter.md:85`
**Issue:** The `hard-cap` rule (and the at-the-cap branch of `circling`) says to record what the fix must be as a decision and accept, "so the run's own agents carry it out".
At the hard cap `continue` is refused and an accept ends the review segment, so no fixer or reviewer round follows; the open design-wrong finding is accepted over and the recorded decision is never acted on.
The brief's rule against accepting over an open design-wrong finding without an operator decision is the guard this rule overrides.

### [MEDIUM] Prose still calls a decision finding the run's parent's alone

**Location:** `contracts/stencils/bouncer/bouncer-fragment-decision.md:7`
**Issue:** The fragment is the `cause_detail` of the brief the arbiter now reads, and it still says each finding is a question only the run's parent can settle and is put to the operator.
That contradicts the changed `bouncer-template-escalation.md` line and the arbiter stencil's `decision` rule.
The same claim stays in `internal/shedadapters/doc.go:108`, `internal/loomcli/circling.go:300` and `contracts/specs/shed-recipe-spec.md:61`, while the `EscalationCause` comment in `internal/shedadapters/escalation.go:18` was changed.

### [MEDIUM] The new notice's non-success paths and its text are unpinned

**Location:** `internal/sheddriver/drive_test.go:452`
**Issue:** The new row covers an arbiter resume at an awaiting stop with an orch recorded.
No row runs the `OrchRecorded` false or error branch, or the failed `Notify` branch, of `noticeArbiterSettled` (`drive.go:565-577`),
no row has an arbiter resume at a blocked stop with an orch recorded, so the `State == awaiting` guard at `drive.go:554` can go without a failure,
and the harness only counts `Notify` calls (`drive_test.go:947`), so the line's content is asserted nowhere.

### [LOW] The decision rule ignores the refusal check on an accept

**Location:** `contracts/stencils/loom/loom-template-arbiter.md:92`
**Issue:** `decisionRefusal` (`internal/shedadapters/bouncerdecision.go:191-196`) can hold an arbiter's accept: it re-judges the round, re-escalates below the hard cap, or halts with `DecisionHaltPrefix` at the budget at or past the cap.
The `decision` rule never points the arbiter at the brief's Refusal section, so a held accept costs another spawn and can end as a parent-only decision halt after the notice said nothing was left to answer.

### [LOW] The stencil's override of the brief's steps is unspecific

**Location:** `contracts/stencils/loom/loom-template-arbiter.md:60`
**Issue:** One sentence says the arbiter's prompt wins where the brief's steps differ, with no list of which brief rules it replaces.
The brief (`bouncer-template-escalation.md:35-46`) still says to settle only on a recorded operator decision, never to count a `--by arbiter` entry, never to accept over an open design-wrong finding, and to report back to the parent, so which rules still bind the arbiter is left to it.

## Rejected

- Code-tests NIT on asserting the notice line, listed apart from its MEDIUM finding: folded into F4, since both are rows of the one notice test.
- Code-tests LOW on the unpinned awaiting-only guard: folded into F4, same test and same fix.
- Code-affected-set finding 3, the `shedadapters/doc.go:108` disagreement with `escalation.go:18`: duplicate of F3.
- Code-affected-set finding 4, the notice sent before the consume: duplicate of F1.
- Code-tests item 8, `Check` returning the brief path for a `decision` escalation: predates the diff and no behavior of the diff depends on it.
- `contracts/stencils/loom/loom-rubric-discussion-review.md:66,71` as stale prose: it defines the `decision` finding class for the discussion review and says such a finding never goes to the fixer, which still holds.
