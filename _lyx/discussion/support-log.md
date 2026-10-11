# Support log — dbench-ohom

## Interview

### Exploration (chair, before the batches)

- Board entry read; the `dbench-*` entries are benchmark copies of this task.
  Related entries checked: `proc-child-lifetime`, `gate-test-hygiene`, `weaver`; no conflict.
- Commit 5a8be0274 (#496) already stopped every shell expiry and records ended shells; the remaining gap is that a gated waiting turn end with both outputs present still counts at once on a fresh first arrival over unawaited shells.
- `gateRepromptText` in `shuttleengine/gate.go` tells every gated agent "fix every finding", the Go-side cause of item 2.
- `recordedSessions` audits Master sessions only, so no recovery session is ever attributed.
- `recover-batch` returns stuck and dead digests through `output.Ok`.
- `lyx quarry resolve` returns `found` for unexported and `_test.go` members today, and `ambiguous` for `internal/proc#StartTime` (per-GOOS).

### Batch 1 (items 1–4) — sent to ly:dbench-ohom:discussion-advisor-1

- Q1, gate shell at Master's turn end: recommended adding `lyx gate test` to Master's `AwaitedShellPrefixes`.
  ly:dbench-ohom:discussion-advisor-1 agreed; noted the shells are most likely forks' shells surfacing on Master's session, which the prefix covers, and that no bound below `master_timeout_min` applies, so the docs must say it.
  It rejected demoting the outcome, since the verify gate still runs the full plan verify.
  Settled as D1.
- Q2, Master self-fix: recommended a caller-supplied re-prompt on `GateEntry`.
  ly:dbench-ohom:discussion-advisor-1 agreed; asked to keep the `Gate findings recorded at` opening fixed and tested, and to add a fixer line atop the verify-gate report.
  Settled as D2.
- Q3, recovery attribution: recommended recording recovery session ids per batch.
  ly:dbench-ohom:discussion-advisor-1 agreed; pointed out the id must be read at spawn through `FindRun` (a done recovery removes its run dir) and that recovery writes must not land in `RunWrites.Master`, which `contractevidence.go` reads.
  Chair confirmed the reader and chose a third slice.
  Settled as D3.
- Q4, recover-batch exit: recommended non-zero for every non-done terminal digest.
  ly:dbench-ohom:discussion-advisor-1 agreed; flagged it as a webster refusal needing a way forward, spec row and test, and that Master's rungs need only one sentence.
  Settled as D4.
- Q5, allowed gates: recommended one statement in the shared implementer body.
  ly:dbench-ohom:discussion-advisor-1 agreed; corrected that the tmux rule is plan policy, not a settings deny, and must name no raw `go` command.
  Settled as D5.

### Batch 2 (items 5–6) — sent to ly:dbench-ohom:discussion-advisor-1

- Q6, build-exclusive ambiguity: recommended one planglyph chokepoint rewriting such an ambiguous answer to multipart for every consumer.
  ly:dbench-ohom:discussion-advisor-1 agreed; required the context set to be a superset of real contexts (atoms plus unnamed unix and non-unix GOOS fallbacks, free release and cgo tags), recommended `Context.MatchFile` with an injected `OpenFile` over hand-parsing, named consumer effects (deleteorder, found-only `renameDeclSource`, create-already-exists wording) and the `x`/`x_test` pair as a fail-closed row.
  Chair added: a candidate file no tried context matches stays ambiguous.
  Settled as D6.
- Q7, Create reusing a name the same card deletes: recommended not building it.
  ly:dbench-ohom:discussion-advisor-1 agreed: the pair contradicts DoneChecks by construction, and D6's Edit-by-glyph route replaces the workaround.
- Q8, refusal text: recommended naming key, status and per-status detail.
  ly:dbench-ohom:discussion-advisor-1 agreed; asked to phrase it as a claim about the lookup, apply it to `rename-not-done`'s new side, and leave `delete-not-done` alone.
  Settled as D7.
- Q9, `lyx quarry resolve`: recommended keeping the raw answer.
  ly:dbench-ohom:discussion-advisor-1 agreed: quarrycli's help promises never to re-shape an answer.
- Q10, testing: ly:dbench-ohom:discussion-advisor-1 added rows (`x`/`x_test`, unconstrained against `tmux || llm`), extending `TestDoneCheckVerdicts_Rules` and the wait table, asserting the default re-prompt for a nil text, the test budget, and hermetic git.
  Folded into Acceptance criteria and Constraints.

## Rejected alternatives

- Item 1, shuttle-wide awaited `lyx gate test` for every gated row: widens a shared contract with no evidence from other rows.
- Item 1, demoting a done run to stuck when a gate shell ended unread: a false negative, since the verify gate's plan verify passed.
- Item 1, lyx stopping the shell itself: the provider reports no pid, so lyx cannot kill it.
- Item 2, a webster verb that spawns the verify fixer: forks are in-session Agent spawns; a verb would spawn a cold strand, duplicate recover-batch and lose Master's context.
- Item 3, a second recovery starting from a clean tree with the first diff as reference: discards work and needs a destructive step lyx would have to run itself (`PATTERN-no-denied-recovery`, the fabric destruction chokepoint).
- Item 3, reading a prior recovery's session id from its run.json at the next spawn: a done recovery removes its run dir.
- Item 3, filing recovery writes under `RunWrites.Master`: contract evidence would count a recovery's write as Master's.
- Item 3b, non-zero exit only when no `recovery_retry` is offered: ties the exit code to retry state, which the board did not ask for.
- Item 3b, `ok:true` envelope with a non-zero exit: a new shape with no precedent in the repo.
- Item 6, DoneChecks alone accepting build-exclusive ambiguity: leaves #507's plan-gate refusal of Edit and Delete by glyph.
- Item 6, teaching quarry build constraints: quarry is an external module, outside this repo.
- Item 6, hand-parsing filename suffixes with a hard-coded GOOS list: reimplements the go command's rules and pins a perishable list.
- #507 second half, a Create reusing a name the same card deletes: contradicts DoneChecks (Delete wants `not_found`, Create wants found) and needs a new done rule; D6 makes it unnecessary.
- Normalizing `lyx quarry resolve`'s output: breaks quarrycli's no-reshape contract.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 gate shell hold — resolved, advisor-informed (ly:dbench-ohom:discussion-advisor-1): D1.
- Q2 re-prompt text — resolved, advisor-informed (ly:dbench-ohom:discussion-advisor-1): D2.
- Q3 recovery attribution — resolved, advisor-informed (ly:dbench-ohom:discussion-advisor-1): D3, third `RunWrites` slice.
- Q4 recover-batch exit — resolved, advisor-informed (ly:dbench-ohom:discussion-advisor-1): D4.
- Q5 allowed gates — resolved, advisor-informed (ly:dbench-ohom:discussion-advisor-1): D5.
- Q6 build-exclusive ambiguity — resolved, advisor-informed (ly:dbench-ohom:discussion-advisor-1): D6.
- Q7 Create reusing a deleted name — resolved, advisor-informed (ly:dbench-ohom:discussion-advisor-1): not built.
- Q8 done-check detail — resolved, advisor-informed (ly:dbench-ohom:discussion-advisor-1): D7.
- Q9 quarry CLI normalization — resolved, advisor-informed (ly:dbench-ohom:discussion-advisor-1): stays raw.
- Q10 testing — resolved, advisor-informed (ly:dbench-ohom:discussion-advisor-1): Acceptance criteria.
- Free-tag cap of 12 per pair — auto-pick, advisor-informed (ly:dbench-ohom:discussion-advisor-1 called it fine): 2^12 contexts per pair is cheap and covers real constraints.
- A candidate file no tried context matches — auto-pick: stays ambiguous, so a never-building file cannot make a pair read disjoint.
- Provenance of the #505/#509/#517 shells (Master's own or forks') — open, auto-pick: not confirmed from a run's `EndedShells`; D1 covers both.
- Why the quick-recipe recovery could only compile-check tmux steps — open: not visible in code; D5 states the rule regardless.
- Reviving a dead batch through `goto` (#511 point 3) — out of scope, auto-pick: not in the board entry.
