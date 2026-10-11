## Interview

The session ran autonomously, so each turn below is the writer's own exploration and pick, not an operator exchange.

1. Read the board entry and the neighbors (`fork-wait-noise`, `audit-reader-globs`); the sibling `twb-*` entries are bench copies of this task and were ignored.
   Read all seven cited issues with `gh issue view`.
2. Item 1: read `wait.go`, `config.go`, `spec.go` and Webster's `runlevel.go`/`triage.go` at base.
   Found the non-expiring wait and `EndedShells` record already landed; the remaining fault is the counts-at-once path for unawaited shells on a gated run with files present.
   Picked an awaited `lyx gate test` prefix; dropped the explicit-stop branch because lyx holds only a shell id.
3. Item 2: read the Master stencil and `gate.go`.
   Found `gateRepromptText` says "fix every finding", which contradicts the stencil; picked a per-entry re-prompt line for the `verify` entry.
4. Item 3: read `recoverbatch.go`, `runwrites.go`, `state.go`, `reset.go` and `contractevidence.go`.
   Found `loadRunWrites` audits Master sessions only, so no recovery edit is ever evidence; `BatchState.SessionID` is the Master session, empty for a recovery.
   Picked recovery session ids on the record and a third write field.
   Read the CLI verb: a terminal `stuck` or `dead` digest returns `output.Ok`, hence exit 0.
5. Item 4: read the recovery prefix and the shared implementer body; the body bans only mutating `lyx fabric` verbs and routes module-wide and tmux runs through `lyx gate test`.
6. Items 5 and 6: read `donecheck.go`, `create.go`, `resolve.go`, `plangate.go` and quarry v0.2.1's `resolve.go`, `golang.go` and `walk.go`.
   Quarry indexes every declaration whatever its case or file kind and reports ambiguity without build constraints by design.
   Picked a lyx-side disjointness predicate shared by DoneChecks and the plan validator.
7. Read the pattern background files that touch the decisions (`leaf-packages`, `stencil-ownership`, `refusal-way-forward`, `completion-signal`, `friction-capture`); no entry is changed.

The last turn of this section holds the lens review:

Lens review, six forks, all returned a report:

- premises, MEDIUM, reset widening justified by "reset stops every recovery strand": fixed, restated per all targets as a recovery being the run's own writer.
- premises, MEDIUM, predicate vacuous for one candidate or a unit collision: fixed, at least two candidates in two files.
- premises, LOW, Master stencil rung wording: fixed, in Decision 3.
- premises, LOW, in-flight records lack session ids: fixed, in Open risks.
- premises, LOW, `PATTERN-gate-slot` cited wrongly: fixed, bullet dropped.
- premises, LOW, other consumers treat ambiguous as unreadable: fixed, named in Decision 6's bound.
- goal-scope, MEDIUM, Create unflaggable at Plan time and relaxation licensed by #507: fixed, stated in Decision 6.
- goal-scope, MEDIUM, wrapped gate commands still end `done`: fixed, Goal narrowed, bound and Open risk say so.
- goal-scope, MEDIUM, ladder stencil edit and refusal class missing: fixed, in Decision 3.
- goal-scope, MEDIUM, exit-0 reinterpretation only an assumption: fixed, added to Open risks.
- goal-scope, LOW, stencil sentence could license Master running tests: fixed, worded as a constraint on shells Master starts.
- goal-scope, LOW, unset custom-tag cap: fixed, eight.
- goal-scope, LOW, quarry reason: fixed, wording below; "likely" for the incident reprompt: fixed, an assumption.
- board-fit, MEDIUM, `fork-wait-noise` shared Master stencil: fixed, Constraints.
- board-fit, MEDIUM, `proc-child-lifetime` beneficiary: fixed, Constraints.
- board-fit, LOW, `verify-watchdog`, `burler-multi-llm`, `parent-review-hardening`: fixed, named in Open risks and Constraints.
- board-fit, LOW, `plan-rubric-file-target`, `plan-fix-card-delete`, `webster-parallel-execution`, `weaver`, `audit-reader-globs`: rejected, no shared decision; a file overlap is the plan writer's merge concern.
- pattern-fit, MEDIUM, refusal class unassigned: fixed, one row per kind with classes.
- pattern-fit, MEDIUM, friction note unverified: fixed, premises fork confirmed `noteRefusals` keys on `ok:false`; acceptance criterion added.
- pattern-fit, MEDIUM, `glyph-build-split` spec homes: fixed, plan spec check-ID list and `planglyph` `doc.go` named.
- pattern-fit, LOW, heading coupling, no-denied-recovery scan, gate-slot: fixed.
- scenarios, MEDIUM, predicate vacuity and unparseable constraint: fixed.
- scenarios, MEDIUM, Edit of a split member downstream: fixed, drift shares the status policy, scope guard reads candidates, acceptance criterion added.
- scenarios, MEDIUM, absent-transcript tolerance needs a typed seam condition and a relaunched strand may use another session: fixed, in Decision 3 and Open risks.
- scenarios, MEDIUM, ladder wording and ordering of persistence: fixed.
- scenarios, LOW, resumed Master with a gate shell: rejected, `countsAtOnce` already returns false for a non-fresh gated run; a finished shell with no wake is added to the bound.
- scenarios, LOW, tmux-only card gates: fixed, in Decision 4's bound.
- scenarios, LOW, path-only attribution: rejected, the same conservative class as Bash writes.
- leanness, MEDIUM, rejected alternatives inside Decisions: fixed, moved out.
- leanness, LOW, duplicated assumptions, restated patterns, negative constraints, tallies, scope restatement, doc-sync criterion, #507 duplicate: fixed.
- leanness, LOW, perishable specifics: fixed, the version and counts removed.

## Rejected alternatives

- **A Master verb that spawns the verify fixer (item 2).** An in-session fork exists only through Master's Agent tool, and a cold strand loses the inherited context the fork is chosen for.
- **lyx stops the gate shell and records the discard (item 1).** lyx sees a shell id and label, not a pid; the strand removal already ends such shells, and the record exists.
- **Awaiting every shell (item 1).** A stray shell a fork left after finishing would hold the run to its deadline.
- **Awaiting by substring or by a config key (item 1).** Shuttle's seam is a label prefix; a substring match adds false holds, and a config key adds a surface for one fixed verb.
- **Second recovery starts from a clean tree with the first attempt's diff as reference (item 3).** It discards the unfinished work, and removing it falls under fabric's destruction chokepoint.
- **Keep attribution by shuttle run id (item 3).** The id was never the cause; the recovery session was never read.
- **Exit non-zero only for `batch_failed` (item 3).** It already does; the exit-0 cases are the stuck and dead digests.
- **Make quarry honour build constraints (item 6).** The board entry allows it, but quarry's engine deliberately does not interpret GOOS, GOARCH or tags, and a change there lands in a separate module bumped in `go.mod`; the lyx-side predicate is local and testable.- **A blocking Plan-time flag for split members (item 6).** It would contradict DoneChecks accepting them, and a not-yet-declared split Create shows nothing to flag.
- **Fix the re-create-a-deleted-name half of #507.** Not in the entry's Expected; a separate resolve-order question.
- **Extend the fork prefix with the gates clause (item 4).** Forks inherit Master's context; no fork reported the doubt.

## Review rounds

_No rounds yet._

## Question ledger

| Question | Resolution |
|---|---|
| Does the gate-shell cut-off still exist at base? | Resolved by reading: only the counts-at-once path for unawaited shells with files present. |
| Hold the shell or stop it? | Auto-pick: hold, via the awaited prefix; stopping needs a pid lyx lacks. |
| Verb or message for the verify fixer? | Auto-pick: message; a verb cannot spawn an in-session fork. |
| Which attribution rule for recovery files? | Auto-pick: by batch, through recovery session ids on the record. |
| Which exit does `recover-batch` give a terminal stuck or dead? | Auto-pick: non-zero with the digest fields kept. |
| Do unexported or test-file Create targets resolve? | Resolved by reading quarry v0.2.1: yes; pinned by a test. |
| What does "flag at Plan time" mean for a split member? | Auto-pick: informational `glyph-build-split` for existing members; a not-yet-declared Create cannot be flagged. |
| Is the cited `rusageStats` failure a resolver limit? | Open: not determinable from the issue; the detail now lets the next occurrence say. |
| Is the second half of #507 in scope? | Auto-pick: no, the entry's Expected omits it. |
