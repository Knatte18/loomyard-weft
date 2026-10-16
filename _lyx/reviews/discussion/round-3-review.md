---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:D2.8
    summary: D2.8 has Go write a section named Verify gate fixes while the Master's step 4 also writes a section of that name, with no rule for which one the heading belongs to
    fix: In D2.8, name the heading Go appends and make it differ from the Master's step-4 heading. Go appends `## Verify gate fix commits`, and the master stencil's step 4 renames its own section to `## Verify gate findings`, still naming only the findings from the findings file. Change the matching Acceptance criterion ("After a fix the verify gate passes on, `summary.md`'s `## Verify gate fixes` section names each `fix:` commit's SHA and subject ...") to use the Go-written heading. Add one clause to D2.8 saying Go appends the section after the Master's last rewrite of `summary.md`, and appends nothing when the notes hold no fix commit.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D2.8
    summary: D2.8 keeps the verify-fix fork's final reply, which names its commits, as an input to the Master while saying the Master never takes fix data from that reply
    fix: In D2.8, add that `webster-body-verify-fix.md`'s "Your final action" no longer asks the fixer to name each commit it made, and that the reply names only a failure the fixer could not fix. The Master's outcome rewrite needs that failure. Add `webster-body-verify-fix.md`'s final-action paragraph to the wording changes D2.2 and the Scope list already assign to that stencil.
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D2.8
    summary: D2.8 records at each gate evaluation but does not say whether a commit seen in several evaluations is kept once, nor which commits of the first-parent range count
    fix: In D2.8, add that `VerifyGateNotes` keeps each commit once, keyed by SHA, the way `addFlaky` keeps each identity once. Add that the trailer read covers the range's non-merge commits only, as D2.3 does, so a clean parent merge in the range is neither listed as a fix commit nor read for trailers.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D5.4
    summary: D5.4 cites the Master's orientation claim as "at :31" without a file, a line number that moves with any edit above it
    fix: In D5.4's last paragraph, replace "The webster master's orientation claim at `:31`" with "The sentence of `webster-template-master.md` that says the Master's reads stay current for the whole run because the run refuses a drifted plan", and keep the narrowing to the plan the drift check covers.
---

### [MEDIUM] Two writers for one summary section

**Location:** `_lyx/discussion/decision-record.md` D2.8 and its matching Acceptance criterion; `contracts/stencils/webster/webster-template-master.md`, step 4 of "A gate failure: spawn one fixer fork".
**Issue:** D2.8 moves the fix commits and departures to Go, which writes them "into `summary.md`'s `## Verify gate fixes` section" after the wait.
It also keeps a Master-written step-4 section, and the stencil today names that section `## Verify gate fixes` too.
The Master rewrites `summary.md` whole after each fixer, while Go only appends (`appendToSummary` opens with `O_APPEND`).
Go's write therefore lands as a second H2 of the same name, or the plan writer has to invent a merge rule.
`summaryparser.Parse` checks only the title heading, so nothing catches a duplicate.
The design's other appended sections (`## Integration suite triage`, `## Plan rebaselined`) each have their own heading, so a distinct heading is the existing convention.

### [LOW] The fixer's reply still names its commits

**Location:** `_lyx/discussion/decision-record.md` D2.8; `contracts/stencils/webster/webster-body-verify-fix.md`, "Your final action".
**Issue:** That stencil tells the fixer to end with a reply "naming each commit you made" and says "Merriam reads only that reply".
D2.8 and the master stencil's raw-output rule say the Master takes no fix data from the reply.
The Master still receives the reply as the fork's completion notification, so the stencil invites the data D2.8 routes through Go.
The reply stays useful for the failures the fixer could not fix, which the Master's outcome rewrite uses.

### [LOW] Recording semantics of the gate's notes are open

**Location:** `_lyx/discussion/decision-record.md` D2.8.
**Issue:** The closure runs at every arrival, and `commitsSince(preFix)` returns the whole range each time, so a second evaluation sees the first evaluation's commits again.
The range is first-parent and includes merge commits (`commitsBetween` uses `--first-parent`), while D2.3 reads non-merge commits only.
D2.8 leaves both open, and the Acceptance criterion would pass with duplicate lines or with a merge commit's trailer.

### [LOW] Perishable line reference

**Location:** `_lyx/discussion/decision-record.md` D5.4, last paragraph.
**Issue:** "orientation claim at `:31`" names no file and pins a line number.
In `webster-template-master.md` line 31 is the sentence "These reads ... stay current for the whole run because the run refuses a drifted plan".
Any edit above it, including this task's own, moves the line.

## Verdicts on the focus questions

**Verify-fix commits and departures (D2.8, D2.4).**
Not a defect in the routing; F1 to F3 cover the wording around it.
The gate closure holds the data at every evaluation: `commitsSince(preFix)` and the closure-local `preFix` persist after a passing evaluation, since `pass()` clears only the persisted copy.
A fix after which the gate passes is therefore recorded, and `finishMasterDone` applies `gateNotes` after the wait with no Master turn needed, so the Master never needs the data.
The data reaches the summary only through Go's git read; the only raw-output path left is the fixer's reply, covered by F2.
D2.4's batch departures come from the persisted digest and the progress trail, which a resumed Master holds.

**Malformed-record refusal (D5.4).**
Not a defect.
The way forward names the operator as the actor that edits and commits the record, and `orch-template-role.md` binds only the orch ("Never edit or commit in either worktree of a task pair, `_lyx/plan` and the decision record included").
D5.4, Acceptance criterion "Each pattern directive renders the declared changes ..." and the question ledger's Q13 all name the operator's record fix, `validate-discussion` and the re-step, so the three agree.

**Landing check and a bullet appended after the plan gate (D5.2, D5.3, D5.7).**
Not a defect.
D5.7 reads the record's `## PATTERN changes` as rendered into the directive by D5.4, never the plan's, and the last-but-two Acceptance criterion ("A `## PATTERN changes` bullet `decision add` appends after the plan gate is rendered into every later directive and checked by D5.7's landing check") covers a bullet appended after the plan gate.
D5.3's rationale still says Webster-Review "measures against the plan", which only holds for the plan's mirror of the section.
That sentence is accurate for the operator's stated reason for the plan section and does not conflict with D5.7's source, so I raise no finding on it.
