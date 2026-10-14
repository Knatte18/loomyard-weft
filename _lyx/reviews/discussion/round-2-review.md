---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:83
    summary: D7's finding left open has no mechanism in the Darn-Review perch, whose contract fixes or disputes every finding, so the fixer is left to revert the parent's correction or the perch stalls
    fix: "Rewrite D7's sentence at line 83, and acceptance lines 167 and 168, to name the mechanism. Recommended: the rubric files a deviation the change description names as an unrecorded parent correction as a `decision`-class finding at MEDIUM whose fix text tells the fixer to leave the code and dispute it, naming the parent as the party who records or refuses the correction with `lyx loom decision add`, and the review row's `fasit` instructions say the same. State in D7 what Darn-Bouncer then does with that open MEDIUM entry: the judge's round-2 CONTINUE rule over an earlier-open MEDIUM gating entry and the circling escalation both apply to it, so either D7 accepts that the perch escalates to the parent (and says so, making the parent the one who settles it at the perch), or D7 names the judge rule that lets it accept with the entry open so D8's carry-over files it. Add an acceptance line that the fixer leaves the deviating code in place."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:102
    summary: D9 gives the alignment agent no rule for a carried-over unrecorded parent correction, so it rejects it as a deviation and PR-Gate bounces to Darn-Write, the one session that cannot record it
    fix: "Add a sentence to D9 after line 102: on a darn run, an open risk carried over from Darn-Review that names an unrecorded parent correction is escalated to the parent at `pr-gate`, never approved and never rejected, since only the parent can record or refuse it. Make the told value D9 and D10 describe carry that rule, and add an acceptance line: a darn PR-Align whose record's `## Open risks` holds such a carried-over finding escalates to the parent."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:70
    summary: D4's dropped-entry check and D5's committed-copy check both measure against the record's committed copy, which a writer can move with `lyx fabric commit`, so an appended entry can still be lost without any refusal
    fix: "In D4, change the reference of the darn-run check from the record's committed copy to every `Added after Discussion` heading the record's commit history in the weft has ever held (or the headings in commits whose message is the discussion-artifacts commit Go writes), so a commit of the record from outside Go cannot erase the reference. In D5, add that the verb confirms after its commit that the committed record holds its own entry, and refuses naming the entry when it does not, which covers a writer write between the verb's append and its commit. Restate D5's Bound at line 70 to match, and add an acceptance line: a record committed by `lyx fabric commit` without an earlier appended entry is still refused by the handoff `discussion` gate, naming the entry."
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:108
    summary: D10 makes only the measuring stick a told value, but the stencil's next line defines a scope deviation as a recorded decision the change breaks, which on a darn run turns the writer's own decisions back into the answer key
    fix: "Extend D10 at line 108: the `pr-gate` scope-deviation definition in `loom-template-alignment.md` (today a change the goal does not call for, a goal point left out, or a recorded decision the change breaks) also comes from the told value, so loom's wiring keeps the recorded-decision clause and darn's names only the board entry and its `Added after Discussion` entries, with the writer's own decisions a claim. Extend the acceptance line on the darn PR-Align prompt to cover the deviation definition."
---

### [BLOCKING] D7's finding left open has no perch mechanism

**Location:** `_lyx/discussion/decision-record.md:83`, with acceptance lines 167 and 168.

**Issue:** D7 says a deviation the change description names as an unrecorded parent correction "is never fixed by revert: the review records it as a finding left open, which D8 carries into the record's `## Open risks`".
The perch has no such state.
Burler's review format says every finding of every class is fixed, and `PATTERN-review-round` requires each finding fixed or disputed.
Darn-Burler-Fix runs `fix-scope: source` with no fasit and no rubric in its profile (`contracts/recipes/darn-recipe.yaml`), so a fixer handed a scope finding against the board entry has only the finding text, and its obvious fix is the revert that `f1ce95ce5` made in `webster-pack-last`.
Severity decides the rest, and D7 names none.
`buildCarryOver` in `internal/shedadapters/bouncercarryover.go` carries only open entries at MEDIUM or worse, so a LOW finding never reaches `## Open risks`.
A BLOCKING open entry never lets the Bouncer accept (`internal/shedadapters/doc.go`: a CONVERGED with an open BLOCKING ledger entry is re-judged).
A MEDIUM `design` entry kept open on purpose meets the round-2 CONTINUE rule over an earlier-open gating entry and the circling escalation, so the perch escalates to the parent rather than accepting and carrying it over.
So the path D7 and acceptance line 168 describe, "left open ... and the change stays in the diff", is not reachable as written, and this is the fallback for the exact failure the board entry records.
This settles the third focus question: the review today has no way to hold the deviation open, and the record must say what the rubric, the fixer and the Bouncer each do with it.

### [MEDIUM] PR-Align has no rule for a carried-over unrecorded correction

**Location:** `_lyx/discussion/decision-record.md:102`.

**Issue:** D7 sends the unrecorded correction to `## Open risks` "for the alignment agent and the parent to settle".
D9's rule has the agent measure the diff against the board entry as amended by `Added after Discussion` entries; an unrecorded correction is by definition not among them, so the agent sees a scope deviation and rejects.
D9 routes a reject to Darn-Write, which per D6 may not run `decision add` and per D3 may not edit its record, so its only moves are to revert, the original failure, or to keep the change and wait for the second reject to reach the parent.
Only the parent can settle the claim, and D9 does not tell the agent to escalate it.
On the second focus question the record is otherwise clear: on a darn run with no orch turn the diff is measured against the board entry as amended by the post-writer entries, and the writer's own sections are a claim, apart from the stencil clause F4 names.

### [MEDIUM] The dropped-entry checks key on a copy the writer can commit

**Location:** `_lyx/discussion/decision-record.md:70`, D4 at line 54 onward.

**Issue:** The first focus question asks what mechanically stops a later whole-file write from dropping an appended entry.
The record's answer is D4's check that the record's committed `Added after Discussion` entries are present in the working copy, and D5's refusal while the working copy differs from the committed copy.
Both measure against the weft's committed copy, and only Go is supposed to commit it, yet nothing enforces that.
`_lyx` in a task worktree is a link into the records worktree (`claimed-scope-edit-weft`), and `lyx fabric commit` routes `_lyx` paths to the weft side (`internal/fabricengine/commit.go`); the darn writer's settings deny no `lyx` command, and the decision verb's own commit-failure way forward names `lyx fabric commit`.
A writer that rewrites its record without an appended entry and commits it that way moves the reference, after which D4's check passes and the entry is gone with no refusal.
A smaller window sits inside the verb: `decisionVerb` appends without holding any lock and then `commitDiscussion` commits whatever the directory holds, so a writer write between the append and the commit is committed by the verb itself, which still reports ok.
The weft write lock D5 cites is real (`CommitRecordsPaths` takes it), but it serializes commits only, not the append or the writer's file writes.
The lost-entry case needs the writer to break D3 first, so this is MEDIUM, not BLOCKING; the cure is a reference no non-Go commit can rewrite, plus a post-commit check in the verb.

### [MEDIUM] The deviation definition at `pr-gate` stays fixed

**Location:** `_lyx/discussion/decision-record.md:108`.

**Issue:** `contracts/stencils/loom/loom-template-alignment.md` line 51 names what the run is judged against, and line 52 defines a scope deviation as "a change the goal does not call for, a goal point the change leaves out, or a recorded decision the change breaks".
D10 moves only the first into a told value.
Left as written, line 52 makes breaking any of the writer's own D-entries a deviation on a darn run, so the writer's unreviewed sections become part of the answer key at the landing route that now runs with no orch turn, against D9's own rule and its stated bound.
The plan writer reads only `decision-record.md`, so the record must say the definition moves with the measuring stick.
Round 1's F2 quoted this clause, and its fix carried only the measuring stick into the told value.

## Rejected

- D7 narrows the operator's "holds the record authoritative over the board body" to the post-writer entries without asking: rejected as a finding, since D7 states the narrowing and its reason and the support log's Rejected alternatives records the wider reading; the alignment agent and the parent see it in the record.
- D7 lets an arbiter-written entry override the board body: rejected, since the arbiter's own stencil hands on any call that widens the task's scope, and D7 states the caller rule as the bound.
- D4's refusal names the missing entries but does not carry their text: rejected, since the writer can read the committed copy with read-only git in the records worktree.
