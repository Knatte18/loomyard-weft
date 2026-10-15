---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:67
    summary: D5 does not say whether a pending entry's held time is a gate-kind span or a third kind, so D6 either counts a held parent-review twice (held span plus the evaluations it re-runs inside it) or has no line to draw the held time on.
    fix: In D5, name the span kinds as a closed set of three (gate, held, rework) and state that the evaluations a pending entry re-runs while held record no gate span of their own, the held span covering them. In D6, add the held time to the gate line, for example `gate <name> <gate time>` where gate time is the sum of the entry's gate and held spans, and state that no two spans of one entry overlap. Add an acceptance criterion that a pending entry re-evaluated while held reports one held span and no overlapping gate spans.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:73
    summary: D5 derives its carrier set from the producers that copy a gate count today, so a gated run whose producer copies none stays dark; the Burler review row's review-parse gate (burlerengine's ReviewGateEntry, re-prompting up to its budget) is one such run and D5 neither carries nor excludes it.
    fix: In D5's Carriers paragraph, define the carriers as every row whose producer runs a gated shuttle run, rather than those that copy a count today, and name the Burler review row as a carrier alongside the Webster row; or, if review-row gates are meant to stay out, add a Scope Out bullet naming the Burler review row's gate and the reason, and narrow the first Auto-mode assumption to say so.
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:95
    summary: D7 attributes rework only to a history entry, so the routed-to row's open visit right after a gate bounce (PR-Rework running) is drawn without its rework-for mark and its time is missing from the gate row's rework total until it closes; D7 neither chooses this nor the alternative.
    fix: In D7, state what the open visit draws when the last history entry is a gate bounce routed to the open visit's row, either that it reads ` rework for <gate row>` and the gate row's rework total includes it drawn with FormatMinutes per D1, or that it stays unmarked until its entry is written; add the chosen case to the D7 acceptance criterion.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:58
    summary: D4 takes the child number from the one D3 gives that row, but D3 gives a review segment's rows no number of their own, only the segment block one, so the stage cell for a child in Plan-Burler-Review or Plan-Bouncer has no defined number.
    fix: In D4, after the child-number clause, add that a child row inside a review segment takes its segment's number, for example `6.6 Plan-Burler-Review`, and add that case to the batten overview acceptance criterion.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:58
    summary: D4's examples do not match D3's rule over the shipped recipes, Land is batten's seventh row and Run-Shed its sixth, and Plan-Write is loom's fifth entry once Discussion-Review takes one number, so `5. Land` and `4.7 Plan-Write` would seed wrong test expectations.
    fix: In D4, replace `5. Land` with `7. Land` and `4.7 Plan-Write` with `6.5 Plan-Write`, or mark both as illustrative shapes that no test pins.
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:131
    summary: The third Auto-mode assumption restates D5's rework-span definition (from each re-prompt to the writer's next arrival) instead of an assumption D5 does not already state.
    fix: Delete the third bullet under Auto-mode assumptions.
---

### [MEDIUM] Pending-held time overlaps the gate spans it contains

**Location:** `_lyx/discussion/decision-record.md:67` (D5) and D6.

**Issue:** Shuttle's wait loop re-evaluates a pending entry at every `gateRecheckEvery` while the writer is idle (`internal/shuttleengine/wait.go`, the `gatePending` recheck branch), so a held `parent-review` runs many closure evaluations inside its held time.
D5 lists "a gate span for each closure evaluation, and for a pending entry such as `parent-review` the time held pending" without saying whether the held time is a gate-kind span.
If it is, D6's "sum of that entry's spans of that kind" adds every evaluation inside the held span on top of it, a double count.
If it is a separate kind, D6 draws only gate and rework times and the held time has no line.
The focus asked whether a double count is possible: on D5 and D6 as written, the plan writer can build one.

### [MEDIUM] Carrier set misses gated runs that copy no count today

**Location:** `_lyx/discussion/decision-record.md:73` (D5, Carriers).

**Issue:** `internal/shedadapters/gateattempts.go` serves `MultiLLMProducer` and `BurlerFixProducer` only, and D5 takes those two plus the Webster row as its carriers.
The Burler review half runs gated too: `internal/burlerengine/review.go` passes `GateSpec{ReviewGateEntry(p.ReviewPath)}`, a PassOnCap entry that re-prompts the reviewer up to `reviewGateAttempts` times (`internal/burlerengine/verdict.go`).
The lens round found the Webster gap by the same route, so the method of starting from today's copiers, not from today's gated runs, is what left this one; the first Auto-mode assumption names writer-row gates only and does not say the review row's gate is out.

### [LOW] D7 is silent on the open rework visit

**Location:** `_lyx/discussion/decision-record.md:95` (D7).

**Issue:** The open visit is not a history entry (`checklistVisit.index` is -1 for it in `internal/loomcli/checklist.go`), so D7's "the history entry right after the bounce entry" excludes it.
While PR-Rework runs after a PR-Gate bounce, its line carries no ` rework for PR-Gate` and the gate row's ` rework` total omits it, and both change once the entry is written.
This is decidable, but the Decision does not say whether it is intended, and the acceptance criterion covers only closed visits.

### [LOW] D4 child number undefined for review-segment rows

**Location:** `_lyx/discussion/decision-record.md:58` (D4).

**Issue:** D3 numbers a review segment's block once and says its rounds and bounces carry no numbers.
D4's "the one D3 gives that row" assumes each child row has a number, and the child's `current_producer` is often a Bouncer or Burler row, which D3 numbers only through its segment.

### [LOW] D4 examples contradict D3's rule

**Location:** `_lyx/discussion/decision-record.md:58` (D4).

**Issue:** `contracts/recipes/batten-recipe.yaml` lists Worktree-Create, Seed-Child, Open-Window, Seed-IDE, Open-IDE, Run-Shed, Land, Worktree-Teardown, so Run-Shed is 6 and Land is 7.
`contracts/recipes/loom-recipe.yaml` gives Preflight 1, Loom-Preflight 2, Discussion-Write 3 (as D3's own example says), Discussion-Review 4 and Plan-Write 5.
D4's `5. Land` and `4.7 Plan-Write` match neither.

### [NIT] Auto-mode assumption restates D5

**Location:** `_lyx/discussion/decision-record.md:131`.

**Issue:** D5 already defines the rework span as running from each re-prompt to the writer's next arrival; the assumption adds nothing a plan writer acts on.

## Focus verdicts

- D8 `role_prefix`: not a defect.
  The key is read only from the embedded recipes (`contracts/recipes/recipes.go`), D8 lists its values and pins them with the recipe test, and `agentname.ValidateRole` holds the grammar.
  A prefixed role cannot equal a `NumberRole` form or a `MatchesRole` match of another base, since it ends in a fixed non-numeric suffix, and the existing roles `discussion`, `plan` and `darn` differ from `discussion-bouncer-seed` and its kin.
  The closed value set bounds the name length.
- D5 and D6 spans: double count possible, filed as F1.
- D7 attribution: decidable from history for PR-Gate and darn's PR-Gate to Darn-Write.
  A pause writes no history entry (`persist`'s pause writes record no verdict in `internal/shedengine/run.go`), and `foldsInto` folds only consecutive identical budget-exempt Stucks, so no fold can sit between a gate bounce and the routed-to visit.
  The open visit is filed as F3.
- D3 and D4 numbering: every loom and darn row shape gets a number, a review segment at its first row in recipe order whatever that row is; D4's source for the child number is filed as F4, its examples as F5.
- Support log ledger: every lens fix it lists is present in the record's Decisions, Constraints, Open risks or Notes.
- Scope, Auto-mode assumptions and Open risks: one restatement, filed as F6; no content that belongs outside this Discussion.
