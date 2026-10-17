---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3 Bound
    summary: D3's bound rests on the false premise that the orch's rule is the only route that writes a decision note; an operator-written decision note without the title form no longer moves the snapshot, and the bound never states that miss.
    fix: In D3's Bound paragraph, replace "the orch's rule is the only route that writes a decision note, and it now names the form" with a sentence stating that a decision note the operator writes on the board without the `Decision for <slug>:` title, such as one the bouncer escalation stencil counts as an operator decision by its body, no longer moves the snapshot, so an approve given while such a note is added is applied; and state the bound, that only the title form counts, so an operator who wants the agent's approve held titles the note in that form or records the decision through `lyx loom decision add` when a decision record exists.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Goal, D1 Bound, Acceptance criteria
    summary: The Goal says the landing rows never take a pull request opened before the run started, but D1's zero-bound fallback keeps today's adoption, and the record never says which runs can reach that fallback, so the reader cannot tell whether a reused slug, the failure the task closes, still can.
    fix: In D1's Bound paragraph, after the zero-bound sentence, add that the zero bound is reachable only by a pair whose seed predates the `started_at` field, since the loom and darn bootstrap writes a stamped seed before the landing wiring runs, so a new pair for a reused slug, which gets a fresh stamped seed, never reaches it; and in the Goal's second sentence add the qualifier "for a run whose seed carries its start stamp".
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D2, Constraints PATTERN-refusal-way-forward line
    summary: D2's way forward, close it then resume the run, names neither a verb nor a command, though the Constraint says it follows the `way forward:` clause refusal-spec.md defines, so the plan writer must guess the resume verb and the promised test has no fixed text to pin.
    fix: >-
      In D2, replace "ends in a `way forward:` clause: close it, then resume the run" with "ends in a `way forward:` clause: close that pull request on GitHub, then resume the run with `lyx loom start`", matching the resume verb Publish's push-rejected stop already names; and in the acceptance criterion on an older open pull request, name the same clause.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D3, Constraints PATTERN-producer-pointer-rule line
    summary: D3 copies the title form into the orch role stencil as literal text, pins it with a test and calls that a deliberate PATTERN departure, though the stencil is already rendered with Go data, so the form can be rendered from internal/alignment's declaration with no second copy.
    fix: In D3, replace "The orch role stencil's decision-note rule ... tells the orch to title the note in that form. A test pins the stencil's text of the form to the Go declaration." with "The orch role stencil's decision-note rule renders the title form from `internal/alignment`'s declaration through a template key, as it already renders `{{.command_index}}`, so the form has one declaration."; delete the Constraints line on `PATTERN-producer-pointer-rule`; and replace the acceptance criterion "a test fails when it and the Go declaration diverge" with "the rendered orch role stencil names the decision-note title form".
  - id: F5
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Open risks
    summary: The Open risk names only D2's misfire when the bound moves, but a bound later than the run's own pull request also makes PR-Gate, PR-Align, Finalize's close and the approve and reject verbs find none, and PR-Gate's nil reason then tells the operator to reopen a pull request that is open.
    fix: In the Open risks bullet, after "makes D2 call the run's own open pull request older", add "and makes PR-Gate, PR-Align, Finalize's close and the approve and reject verbs find no pull request, where PR-Gate's existing nil-lookup reason names reopening it, which cannot help".
  - id: F6
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Constraints PATTERN-leaf-packages line
    summary: The Constraints cite PATTERN-leaf-packages for internal/alignment, but that entry's background names no such package, so the line binds nothing and suggests a closed import set the package does not have.
    fix: Delete the Constraints bullet on `PATTERN-leaf-packages`.
---

### [MEDIUM] D3's bound assumes only the orch writes decision notes

**Location:** `_lyx/discussion/decision-record.md` D3 Bound.

**Issue:** The bound justifies the narrowing with "the orch's rule is the only route that writes a decision note".
`contracts/stencils/bouncer/bouncer-template-escalation.md:37` counts "a board note whose body states that it records an operator decision", which the operator writes, normally with no `Decision for <slug>:` title.
Today such a note moves the snapshot when it names the slug (`internal/loomcli/alignmentwiring.go:76-77`).
Under D3 it does not, so an approve given while it is added is applied (`internal/landingshed/pralign.go:264-276`).
The rubric's attack-surface rule requires the bound to state what the narrowed guard now lets through, and this miss is unstated.

### [MEDIUM] The Goal's absolute claim conflicts with D1's zero-bound fallback

**Location:** `_lyx/discussion/decision-record.md` Goal, D1 Bound, the zero-bound acceptance criterion.

**Issue:** The Goal says the landing rows "never take a pull request opened before the run started".
D1's zero bound keeps today's lookup (`internal/landingshed/pullrequest.go:16-33`), which adopts exactly such a pull request, and the record never says which runs reach that bound.
The loom and darn bootstrap writes a stamped seed before the landing wiring runs (`internal/loomcli/sharedbootstrap.go:100-115`, `internal/shedrun/seed.go:175-177`), so only a seed that predates `started_at` gets the zero bound, and a reused slug's fresh pair never does.
With that stated, the warning-only fallback is enough, which settles the focus question; without it, the record leaves open whether the failure the task exists to close still gets through.

### [MEDIUM] D2's way forward names no verb

**Location:** `_lyx/discussion/decision-record.md` D2 and the Constraints line on `PATTERN-refusal-way-forward`.

**Issue:** The Constraint claims the clause `contracts/specs/refusal-spec.md` defines, which names a `lyx` verb, a transition or a `git` command.
"Close it, then resume the run" names none of them.
Publish's push-rejected stop already says "resume the run with `lyx loom start`" (`internal/landingshed/publish.go`, `pushRejectedReason`).
The path is not a dead end: after the close, the bounded lookup finds none and Publish creates a new pull request.
But the stop text is underspecified for the plan writer and for the test the Constraint promises.

### [MEDIUM] The title form gets a second copy and a declared PATTERN departure when it could be rendered

**Location:** `_lyx/discussion/decision-record.md` D3 and the Constraints line on `PATTERN-producer-pointer-rule`.

**Issue:** The record states the form literally in `contracts/stencils/orch/orch-template-role.md`, adds a test pinning it to Go, and calls this a deliberate departure from `PATTERN-producer-pointer-rule`.
The orch role stencil is already rendered with Go data (`{{.command_index}}` at line 16, `{{.repository_rules}}` at line 247).
Rendering the form through a template key from `internal/alignment`'s declaration leaves one declaration and needs no drift test.
It also removes the departure, which the record justifies only by the board having no format contract to point at.

### [LOW] The Open risk misses the other callers of a moved bound

**Location:** `_lyx/discussion/decision-record.md` Open risks.

**Issue:** D1 makes every caller pass the bound.
A bound later than the run's own pull request makes PR-Gate, PR-Align, Finalize's close and the approve and reject verbs all find none, not only D2's Publish stop.
PR-Gate's nil branch then tells the operator to "reopen the pull request or abandon the task" (`internal/landingshed/prgate.go:99-100`), which cannot help while the pull request is open.

### [LOW] PATTERN-leaf-packages does not govern internal/alignment

**Location:** `_lyx/discussion/decision-record.md` Constraints.

**Issue:** `pattern/PATTERN-leaf-packages.md` lists its packages under its own headings, and `internal/alignment` is not one of them; the package already imports `shuttleengine`, `stencilstore` and others.
The constraint binds nothing and reads as if the package had a closed import set.

## Focus settlements

- D1 bound: see F2. The fallback is reachable only for pre-`started_at` seeds, and the warning suffices once the record says so.
- D1 premise: the Decisions, the Auto-mode assumption and the Open risk agree that only a skipped cleanup or a mid-run seed reset can make the stamp an earlier run's, and no acceptance criterion depends on an unstated premise; F5 widens the risk's effects.
- D2: the bound, open and older only, leaves a path that is not a dead end; F3 fixes the wording.
- D3: the title form, a case-sensitive prefix of slug plus colon on kind note, is close enough to implement and test, since `boardengine` stores titles unchanged; F1 fixes the bound.
- D3 and D4 placement: D4 holds; F4 and F6 fix the Constraints.
- First-page assumption: holds for the run's own pull request, which sorts first; D2's older open pull request falls off the first page only with 30 or more newer pull requests on the same head and base, which is not a realistic case, so no finding.
- Relocation pass: no Decision or Scope line carries deliberation that belongs in the support log, and every Scope item serves one of the issue's expectations or the `decisionRecordedReason` duplicate the entry names.

## Rejected

- existing-paths, D1 hard-codes `SelfRunID` while loomcli accepts a run-id: the landing deps are wired in the shared bootstrap, which reads and writes only the `SelfRunID` seed (`internal/loomcli/sharedbootstrap.go:100,115`), and `lyx loom approve` takes no arguments and runs at the pair, so no landing path reads a non-self run's seed.
- bound-breaker, inherited legacy `_lyx/shed/self/` seed escapes cleanup: no such directory exists in the records tree, and the cited path is the one the assumption names; the legacy segment is a migration remnant, not a reachable reuse path.
