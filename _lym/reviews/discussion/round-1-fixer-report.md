# Fixer report — discussion round 1

Every finding's premise held against the code, and every finding is fixed in `_lym/discussion/decision-record.md`.
The support log's Review rounds section records the round, and its Q10 ledger line now matches D2.
`lym loom validate-discussion` answers `ok: true`.

## Changes

- F1 (BLOCKING), fixed as proposed.
  D2 now wraps every other entry of a row that carries a fixer entry to wait on a live fixer record before running its own closure, so a restart's finalize judges every entry over the fixer's finished tree.
  The first Constraints fact names `loomshed.NewCleanGate` as the reason, beside the pending case.
  The D2 attach acceptance bullet names the Darn-Write case.
- F2, fixed as proposed.
  D1 marks records terminal in the store.
  D2's probe answers true only for a live record saved before the last attach.
  D8 keys a row's record set to the row visit.
  A new D2 acceptance bullet says a terminal record alone does not finalize.
  The self-check later moved the terminal mark after the re-run; see Self-check.
- F3, fixed as proposed, first option: D2's Bound now states the real overrun of two fixer timeouts plus four inner gate runs, and the D2 finalize acceptance bullet matches it.
- F4, fixed as proposed.
  D4 step 3 joins any tree in the fixer phase, then rewrites the recorded tree identity when the re-run starts.
  D4's Bound states what the fixer-phase join lets through.
- F5, fixed as proposed.
  D8 moves the agent-call record into the key directory, D4 step 1 attaches to or stops a live recorded fixer, D4's Bound covers a dead runner, and a new D4 acceptance bullet covers the killed runner.
  The attach happens before the new run's gate run, not only before its spawn, so the gate never judges a live fixer's half-done edits.
- F6, fixed as proposed.
  D7 names Plan-Write's lens procedure with its `SelfCheckCommand` and `FinalSelfCheckCommand` rows, and the D7 acceptance bullet covers both lens procedures.
- F7, fixed as proposed.
  D5's leave-list always leaves the lens review record to the writer.
  The Plan-Write scope excludes that record, the re-check names `lym loom validate-plan --lens-review` for a forked run, and the D5 acceptance bullet names the record as the writer's.
- F8, fixed as proposed: D5 has a Bound paragraph with what the edge lets through, what judges it per caller, and which limits Go enforces against those that rest on the prompt.
- F9, fixed as proposed: D7's Webster fork text depends on the `card` key's `fixer`, and a D7 acceptance bullet covers the fixer-off case.
- F10, fixed as proposed: D4's Bound grounds the hand-off in `frontDecision` and says the front writes the parsed role into the run request; the D4 acceptance bullet says "a call that is not capped".
- F11, fixed as proposed: D4's last Bound bullet says Go bounds one fixer per red call, and that the number of red calls per card rests on the fork stencil's cap.
- F12, fixed as proposed: D6 names the runproc registration of webster's `verify-fixer` strand, and the D1, D6 acceptance bullet names that one change.
- F13, fixed as proposed.
  The gate run records in `_bolt/.lym/gate/runs`, grouped by call id, show the longest whole call at 594 s, a `darn-write` call.
  That is recorded as a Constraints fact, 30 is kept as the default, and the assumption line is dropped.
  No `card` call is recorded, so an Open risks line says the card key is unmeasured.
- F14, fixed as proposed: ` (removed)` now marks `internal/websterengine/coldfixer.go` in D1 and D6, `awaitColdFixer`, `contracts/stencils/webster/webster-prefix-verify-fix-cold.md` and the Notes' `RenderVerifyFixColdPrompt`.

## Self-check

Four forks returned reports; none was missing.

- fix-consistency, LOW: Scope Out still listed Darn-Write's `clean` and `description` gates as untouched. Fixed as proposed: the bullet says they get no fixer and only wait on a live one.
- fix-consistency, LOW: D8 put the fixer's time in the fixer entry's gate span. Fixed as proposed: it shows in the span of the entry that waited.
- fix-way-forward MEDIUM and fix-proposal LOW, merged: a fixer that ends leaving uncommitted edits makes a restart halt on `clean` without its reply. Fixed as proposed: D2 says the first entry that fails over a waited-on fixer carries its reply or dead note, and the D2 acceptance bullet is narrowed to match.
- fix-way-forward MEDIUM and fix-proposal MEDIUM, merged: an exact role test misses numbered fixer roles such as `gate-fixer-2`. Fixed as proposed: D4's Bound tests the role as `agentname.MatchesRole` does, and the D4 acceptance bullet names numbered roles.
- fix-claims MEDIUM and fix-proposal MEDIUM, merged: D5's Bound read the later review rows as a Go guarantee. Fixed as proposed: Go runs the rows, and whether they review rests on hub config such as `code_max_bounces`.
- fix-proposal, MEDIUM: marking the record terminal when the wait ends left a restart during the re-run stalled until the deadline. Fixed as proposed: a row or card caller marks the record once it holds a result over the finished tree, and webster marks its own when the wait ends, as today.
- fix-proposal, LOW: a forked Plan-Write gate failing only on the missing lens review still spent a fixer. Fixed as proposed: D5 says that evaluation spawns no fixer, and the D5 acceptance bullet says so too.
- fix-proposal, LOW: the plan fixer can edit files after the lens review read them. Fixed as proposed: D7 states the staleness is accepted and points at D5's Bound.

## Disputed

None.

## Deferred

None.
