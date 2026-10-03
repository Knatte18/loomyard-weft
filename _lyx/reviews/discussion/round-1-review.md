---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md D5, D6 and Open risks
    summary: A finished reflection is one whose report exists, but the stencil makes the report mandatory even when filing failed, so D5 and the clean-return path archive unfiled notes as reflected and D4 never alerts; Open risks misfiles GitHub-auth failure under failed
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D5
    summary: D5 does not say where the covered-notes record lives or that archiving becomes covered-set-only on every path, so the clean-return whole-directory rename can still archive a note written after the reflection started, against D9
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D1, D3, D6
    summary: D1 removes the operator approval gate and D6 files freeform notes, verb arguments and refusal text to the public Knatte18/loomyard repo, but no Decision bounds what content may reach an issue body
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md D2, D5
    summary: D2's skip-on-held-lock and D5's attach-and-wait sit at different layers, but neither Decision says how they compose or what a killed process leaves behind (released lock, covered record without a report)
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md D7, D8
    summary: D7 and D8 are no-change Decisions restating Scope Out and Rejected alternatives; the one constraint they carry, batten's recipe and teardown unchanged, belongs in Constraints and the rest in support-log.md
---

### [BLOCKING] A report proves the agent reached its report step, not that it filed

**Location:** `decision-record.md` D5 (archive-without-spawn edge), D6, Open risks first bullet.

**Issue:** D5 archives a finished reflection's covered notes without spawning, where "finished" can only mean the report file exists: shuttle classifies `done` on bare existence of the declared output (`internal/shuttleengine/wait.go`), and nothing in Go parses the report (`internal/frictionengine/deps.go`, the "no filed status" comment).
The reflection stencil (`contracts/stencils/friction/friction-template-reflection.md` Step 4) tells the agent to write the report "regardless of what Step 3 produced", so an agent whose `lyx selfreport create` calls failed on GitHub auth writes the report, the run classifies `done`, Reflect archives the notes and returns `reflected`.
So the answer to the focus question is yes: a reflection that failed GitHub auth licenses archiving notes that were never filed, both on today's clean-return path and on D5's new archive-without-spawn edge.
A timed-out or killed reflection leaves no report and keeps its notes, which is correct.
D4 then never fires, because the status is `reflected`, not `failed`.
The first Open risk claims a GitHub-auth failure is a "failed terminal reflection" alerted by D4, which is false against the code, and it hides a silent loss path that contradicts the Goal ("every loom run's friction reaches GitHub").

**Fix:** In D6, make the report a success marker: the agent writes the report only after every issue it decided to file was created, or after deciding nothing is worth filing; on a failed `lyx selfreport create` it ends its turn without writing the report.
Shuttle then classifies `asking`, Reflect returns `failed`, the notes stay, and D4 alerts at a done stop.
In D5, define a finished reflection as one whose report exists under that rule.
Correct the Open risk to match, add the duplicate risk this creates (issues filed before the failure are refiled by the next reflection), and add an acceptance criterion for the failed-filing case.

### [MEDIUM] D5 leaves the covered-notes record and selective archiving undecided

**Location:** `decision-record.md` D5.

**Issue:** D5 says Reflect "records the set of notes a reflection covers" but not where the record lives, and the location is constrained: a `.md` file in the friction directory is scanned as a note (`scanNotes`), and a record outside it is not archived with the notes.
D5 also archives "the notes that reflection covered" on the no-spawn path, but says nothing about the clean-return path, which today renames the whole friction directory (`archiveFrictionDir`).
A note written while a reflection runs (an operator resuming a blocked `run` while its halt reflection runs, the case `reflectFriction`'s lock comment describes) is then archived without being covered, which D9 forbids.
The acceptance criteria ("a note added after that reflection started is reflected by the next spawn") assume selective archiving that no Decision states.
D5 also changes the meaning of a leftover report, today deleted as stale, without saying when a report is still deleted.

**Fix:** State in D5 that the record is a non-`.md` file in the friction directory beside the report, that every archive (clean return and no-spawn alike) moves only the covered notes plus the record and report, and that a leftover report with no record is deleted as stale as today.

### [MEDIUM] No bound on what content reaches a public issue

**Location:** `decision-record.md` D1, D3, D6.

**Issue:** D1 removes the run-only gate whose stated reason was that the agent files public issues itself, and D6 files every distinct lyx problem without approval.
`Knatte18/loomyard` is public (`gh repo view` reports `PUBLIC`).
The notes feeding it are freeform agent text from any repository lyx drives, plus D3's verb arguments and refusal text, plus repair records.
D1's bound (timeout, hardcoded repo) and D3's bound ("files nothing by itself") are both true but neither covers content: D3's notes are local, then D6 publishes what they say.
The rubric's attack-surface rule requires the Decision that removes a guard to state what it now lets through; the approval gate was also the only content check.

**Fix:** Add a content bound to D6: an issue body describes the lyx problem and may quote lyx's own output, verb names and repo-relative lyx paths; it never quotes the task repository's source, diffs or plan text, credentials, tokens, environment values or absolute home paths.
Point D1's bound at it and add a matching acceptance criterion on the stencil text.

### [LOW] D2's lock skip and D5's attach probe do not say how they compose

**Location:** `decision-record.md` D2 Bound, D5.

**Issue:** D2 keeps "a held reflection lock skips the reflection" on the halt path; D5 says Reflect probes for a live agent and waits on it.
They do not conflict once placed: the friction lock (`reflectFriction` in `internal/loomcli/run.go`) arbitrates between live processes and is advisory, so the kernel releases it when a step process is killed; the attach probe runs inside Reflect, under the lock, and covers an agent whose driving process died.
On the halt path a held lock still skips, and on the terminal row the lock is waited on as today.
Neither Decision says this, nor what happens to a covered-notes record left with no report and no live agent: under D5's wording those notes are reflected anew, and any issue the dead agent filed before dying is filed again.
A halt note skipped under a held lock also stays for the next reflection, which no Decision states.

**Fix:** Add to D5: the probe runs inside Reflect under the friction lock; D2's skip and the row's wait stay unchanged; a killed process leaves no lock; a record with no report and no live agent is discarded and its notes reflected again, at the cost of possible duplicate issues.

### [LOW] D7 and D8 restate scope and rejected alternatives

**Location:** `decision-record.md` D7, D8.

**Issue:** D7 decides nothing beyond Scope Out's "Per-agent end-of-session self-report sessions", and its rationale repeats the support log's rejected alternative.
D8's ordering sentence restates D1 and D2, its survival sentence restates the first Open risk, and its #274 rationale is history the support log already holds.
The one thing D8 carries that the plan writer needs, that batten's recipe and teardown stay unchanged, is not stated anywhere else: Scope Out covers only batten's recipe having no friction directory.

**Fix:** Delete D7 and D8, add a Constraint that batten's recipe and teardown are unchanged, renumber D9 to D7, and update the support log's references.

## Not-a-defect verdicts

- **D4's notification edge.**
  At a done stop, the Friction-Reflect row waits on the friction lock rather than skipping, so `skipped` there means Tier 2 is off or there was nothing to reflect on, and needs no alert.
  At a halt, the driver already hands back to the parent, and a skipped or failed halt reflection leaves its notes for the terminal reflection.
  With F1 fixed, `failed` at done covers filing failures too, so D4's single edge is sufficient.
