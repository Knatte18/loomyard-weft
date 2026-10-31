---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md (D4, second paragraph; D4 acceptance criterion)
    summary: The fence one longer than the longest backtick run has no floor of three, so a body with inline code only or no backticks gets no fence and its H2 lines become record sections
    fix: In D4 replace "whose backtick fence is one longer than the longest backtick run in the body" with "whose backtick fence is max(3, one longer than the longest backtick run in the body) backticks long", and extend D4's Bound to match. In the D4 acceptance criterion add a body with no backticks and a body holding inline code only, each holding `## Decisions` and `## Open risks` lines, whose `decision add` append lands under the record's own `## Decisions`.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md (D4, the "writes whichever of the two files is missing" sentence)
    summary: A crash mid-write leaves a torn record that Record-Write never overwrites, and the Darn-Write gate then loops on lym loom resume because the writer may not edit the record
    fix: In D4 state that Record-Write writes each file to a sibling temporary file and renames it into place, so an existing file is always whole. Add to the D4 acceptance criterion that a torn temporary file left by a crash is ignored and the resume writes the file whole.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md (D13, the check paragraph)
    summary: D13 puts its retired-key refusal before ValidateLocal, but PrepareRunSettings runs module.Migrate first, and Migrate's drop list can strip the key silently before the check sees it
    fix: In D13 state that the check runs in PrepareRunSettings on each module's raw object, after moveRunSettings and before module.Migrate, and that the keys are pruned through the RetiredKeys reconcile prune and never added to retiredLoomKeys or darn's Migrate drop list.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md (D6, first and third paragraphs)
    summary: With no parent, D6 gives the writer no action on a Decision for slug note that answers its stop, while D7 makes the record, which cannot hold that answer, the only review key
    fix: In D6 state that, with no parent, a `Decision for <slug>:` note does not settle a stop, and the writer stops again with its stop report naming `lym loom decision add` then `lym loom resume`, so every answer reaches the record. State that the darn writer stencil drops its route of writing a note's answer into the record. Add a D6 acceptance row where a no-parent run with an answering note and no `Added after Discussion` entry halts again.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md (D4, the decision add sentence)
    summary: The darn-only decision add refusals for an uncommitted, changed or entry-dropping record keep a way forward naming the Darn-Write session, which D4 and D6 remove from that route
    fix: "In D4, after the missing-record sentence, add: decision add's darn refusals in internal/loomcli/decision.go for an uncommitted record, a record that differs from its committed copy, and a record missing a historical entry drop decisionWayDarnWriter and name lym loom goto --to Record-Write then lym loom step, so Record-Write commits the pair. Add that refusal and its way forward to the D4 acceptance criterion."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md (D4, the Darn-Write in-flight bullet with no commits)
    summary: The no-commit route claims Drift-Baseline captures the baseline again with the record in place, but DriftBaseline keeps an existing scope record and a passed capture takes no new capture
    fix: In D4's no-commit bullet replace "which captures the baseline again with the record in place of one an agent took with no record to read" with a statement that the scope record and capture from the earlier pass stand, since the capture reads code at the branch point, and that the earlier scope agent read the board entry through its drift subject. Or name the step that removes the earlier scope record so Drift-Baseline records it again from the record.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md (D5, the Darn-Write validate-only guard sentence; D4's claim that the spec reads no fenced line)
    summary: missingSections ignores fences, so with a fenced body holding Open risks the validate-only guard passes a record whose real Open risks heading the writer deleted
    fix: In D4, beside the fencedLines change, add that missingSections in internal/discussionparser/validate.go counts a required heading only on a line outside a fence, by the same CommonMark fence rule. Add to the D5 acceptance criterion a D4 record whose body holds `## Open risks` and whose real `## Open risks` heading is removed, which the validator refuses.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md (D13, the RetiredKeys sentence and Rationale)
    summary: A hub base loom.yaml or darn.yaml that tuned driver_kind, arbiter or tackler keys is dropped silently and pruned, contradicting the rationale that refusing keeps a pinned run from running another driver
    fix: In D13 state the bound. Either the reconcile copies a hub base file's tuned value of each retired key into the hub's shed.yaml when shed.yaml does not set it, then prunes it, naming that copy as the one path. Or state that a hub-tuned base value is dropped and pruned with no carry-over, that a run then takes shed.yaml's value, and that the orch moves each tuned value before landing; add the second form to Open risks.
  - id: F9
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md (D4, the Drift-Baseline and Drift-Await in-flight bullet)
    summary: A darn run parked on Drift-Baseline and resumed without the goto spawns the baseline agent on a decision record that does not exist, and its Stuck names drift's ways forward rather than goto Record-Write
    fix: In D4's Drift-Baseline bullet state the bound for a resume without the goto. The baseline agent reads a missing record and may record a scope from the board entry alone or return Stuck with drift's ways forward. Darn-Write's missing-record refusal is what sends the run back to Record-Write. Add this case to the matching Open risks line.
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md (D10 Bound; D10 acceptance criterion)
    summary: D10's Bound says a Preflight run with history past Preflight, a set start_sha or pause_requested now proceeds, but no acceptance row pins what the dropped checks let through
    fix: Add to the D10 acceptance criterion that a run whose current row is Preflight after `lym loom goto --to Preflight`, holding history past Preflight, proceeds into the row after Preflight with no seed-check refusal.
---

### [BLOCKING] Record-Write's fence rule yields no fence for typical bodies

**Location:** `_lym/discussion/decision-record.md`, D4, second paragraph, and the D4 acceptance criterion.

**Issue:** The rule "a backtick fence one longer than the longest backtick run in the body" has no minimum length.
This task's own board body holds inline code only and no fenced block, so its longest run is 1 (checked with `lym board get --body`).
The rule then gives a fence of two backticks, and a body with no backticks gets one backtick.
CommonMark needs at least three to open a fence, and `fencedLines` (`internal/discussionparser/append.go:349-361`) tests `HasPrefix(trimmed, "```")`.
Every body line then reads as record text.
A body `## Decisions` is the first unfenced `## Decisions` `decisionsInsertOffset` (`append.go:307-347`) meets, so `decision add` lands inside `## Goal`.
`locateOpenRisks` (`carryover.go:234-256`) and `CheckDecisionIDs` misread the body the same way.
D4's purpose, and its acceptance row for a body holding `## Decisions` and `## Open risks`, fail for exactly the bodies Record-Write will see most.
All four lenses and the handler found this independently.

### [MEDIUM] A torn record write becomes a permanent stall

**Location:** `_lym/discussion/decision-record.md`, D4, the sentence beginning "It writes whichever of the two files is missing".

**Issue:** The record states no atomic write.
Suppose lym dies while writing `decision-record.md`; the file left behind is truncated.
On resume, Record-Write sees that the file exists, keeps it and commits it.
Darn-Write's validate-only `discussion` gate (D5) then reports missing sections and re-prompts a writer that D6 forbids to edit the record.
The gate exhausts, and the Stuck carries `darnResumeWayForward`, "run lym loom resume" (`internal/loomshed/darnwrite.go:15`, `93-95`).
That respawns the same failing gate.
`goto --to Record-Write` does not help either, because the row never overwrites.
This is the crash case the focus asked about, and the record's resume promise does not hold against the `discussion` handoff gate.

### [MEDIUM] D13's refusal can run after Migrate has dropped the key

**Location:** `_lym/discussion/decision-record.md`, D13, the check paragraph.

**Issue:** `PrepareRunSettings` (`internal/configreg/runsettings.go:46-87`) runs `moveRunSettings`, then `module.Migrate`, then `checkLocalDocument`.
Loom's `MigrateConfig` ends with `yamlengine.DropKeys(migrated, retiredLoomKeys)` (`internal/loomengine/config.go:709`).
D13 says only that its check runs before `ValidateLocal`, and it leaves the reconcile's prune route for these keys unnamed.
If the implementer prunes them the way loom prunes its other retired keys, Migrate strips `loom.driver_kind` from a profile or `run_settings` before the check sees it.
That is the silent ignore D13's rationale and acceptance row reject.

### [MEDIUM] No-parent `Decision for <slug>:` notes have no defined effect

**Location:** `_lym/discussion/decision-record.md`, D6.

**Issue:** Today's stencil (`contracts/stencils/darn/darn-template-write.md`, Step 3, no-parent bullets) has a no-parent writer that finds an answering `Decision for <slug>:` note write the answer into the record and carry on.
D6 forbids the writer to edit the record, yet still has it read those notes, and names only `lym loom decision add` as the halt's way forward.
An operator who answers the way today's stop report says, with a note and then `lym loom resume`, leaves the writer with no stated action.
If it follows the note, D7's perch, whose only answer key is now the record, reviews against a scope that lacks the answer.
If it stops again, the note-and-resume route loops.
This is the no-parent note case the focus named.
Round 1 raised the same gap, and the round-1 fixer reported it fixed, but D6's text still leaves it open.

### [MEDIUM] `decision add`'s darn refusals still point at the Darn-Write session

**Location:** `_lym/discussion/decision-record.md`, D4, the sentence on `lym loom decision add` before Record-Write.

**Issue:** On a darn run, `decision add` refuses three kinds of record (`internal/loomcli/decision.go:380-395`):
- one not committed at HEAD, which is also the before-Record-Write case D4 cites;
- one that differs from its committed copy;
- one that dropped a historical entry.

Each refusal appends `decisionWayDarnWriter`, "Darn-Write has not finished its record or is editing it…; message the running Darn-Write session".
Under D4, Record-Write writes and commits the record, and under D6 the writer never touches it.
After a crash between Record-Write's write and its commit, the way forward names a session that does not exist and a route that is gone.
That breaks PATTERN-refusal-way-forward.
The record calls this "its existing missing-record refusal" as if it needed no change.

### [MEDIUM] The no-commit in-flight route rests on a false re-capture claim

**Location:** `_lym/discussion/decision-record.md`, D4, the Darn-Write in-flight bullet for a branch with no commits.

**Issue:** The bullet says `goto --to Record-Write` then `resume` "captures the baseline again with the record in place of one an agent took with no record to read".
`internal/driftengine/doc.go:68-69` says otherwise.
DriftBaseline records the scope "when either is missing", and it spawns the agent with a note that the scope is already recorded.
It also says "a set awaited_at and a passed capture take no capture".
On the second pass the first pass's scope record and capture stand.
The stated reason for preferring this route over the step route is false.

### [MEDIUM] The validate-only guard reads fenced headings

**Location:** `_lym/discussion/decision-record.md`, D5, the Darn-Write guard sentence, and D4's claim that the spec reads no fenced line.

**Issue:** `missingSections` (`internal/discussionparser/validate.go:113-129`) matches every line verbatim and never calls `fencedLines`, even though its own doc says a fenced heading never counts.
Take the body D4's acceptance row names, one holding `## Open risks`.
If the writer deletes the record's real `## Open risks` heading, `Validate` still passes on the fenced copy.
D5 says the guard proves "the writer left the record intact", but it lets this through.
The Bouncer's carry-over then fails on `ErrNoOpenRisksHeading` (`carryover.go:234-256`).
D4 changes the fence rule but leaves this reader out of it.

### [MEDIUM] Hub-tuned retired keys are dropped silently

**Location:** `_lym/discussion/decision-record.md`, D13, the RetiredKeys sentence and its Rationale.

**Issue:** D13's own text says a hub base `loom.yaml` or `darn.yaml` value is dropped at every load until the reconcile prunes it, and nothing moves it.
A hub whose base `loom.yaml` tuned `driver_kind: go` then silently runs `shed.yaml`'s driver after landing.
That contradicts the Rationale's "refusing rather than ignoring keeps a run pinned to a driver from silently running another".
Retiring a guard-like override needs its bound stated (rubric: attack-surface question).
This hub's `_board/_lym/config` holds none of these keys, so the impact falls on other hubs, which the decision does not bound.

### [LOW] An in-flight darn run resumed on Drift-Baseline without the goto reads a missing record

**Location:** `_lym/discussion/decision-record.md`, D4, the Drift-Baseline and Drift-Await bullet.

**Issue:** D4 switches darn's Drift rows to `subject: decision-record`.
A run parked before Record-Write and resumed without the goto spawns the baseline agent on a record path that does not exist (`internal/driftengine/seats.go:60-63`).
The record bounds the no-goto case only at Darn-Write.
The run recovers there, but Drift-Baseline's own outcome on a missing subject is unstated.

### [LOW] D10's dropped checks have no acceptance row

**Location:** `_lym/discussion/decision-record.md`, D10 Bound, and the D10 acceptance criterion.

**Issue:** The Bound says what the dropped half-finished and stat-race checks now let through.
No acceptance criterion pins that behaviour, so a plan writer has nothing to test the retired guard's new edge against.

## Rejected

- existing-paths 4, the darn Drift subject set by the wiring's `RecipeDarn` branch at `internal/loomcli/wiring.go:785`: D8 already retires every remaining `RecipeDarn` branch in `internal/loomcli` for the recipe's rows or keys, and D8's acceptance criterion says the wiring holds no recipe-name branch, so the row's `subject` drives it.
- interactions F3 and walkthrough F2 as separate findings: they are the same defect as F2 here, the torn write and the loop on the validate-only gate, and they are merged into it.
- interactions F4 and walkthrough F5: merged into F9.
