---
round: 1
focus:
  - "Cards 10, 11 and 12 (loop body, detach and wait, watchdog): do they state in `**Intent:**` what each new escape or kill path can skip or let through and what bounds it? Look at `--until-stop`, the pid-file kill, the `Detach` fallback when breakaway is refused, the dead-loop stop and the watchdog kill, against D2, D4 and D5 of the decision record."
  - "Card 6 (`pause --before|--after|--clear`) and card 4: does the Intent state the bound on `--clear` and on one actor retargeting another's condition, as D3 does?"
  - "Card 10 sizing: one card carries the loop body, a new `PATTERN-failure-stop-visible`, the `PATTERN.md` overview edit, `refusal-spec.md` rows, `docs/overview.md` and a budget raise. Is this one independently reviewable unit, or should part of it be its own card? Check the granularity rule in both directions."
  - "Card 8 bundles the kind vocabulary, the arming refusal and six unrelated `PATTERN.md` overview rewordings. Is each rewording licensed by the decision record's PATTERN overview cap constraint, and is the bundle a sound unit?"
  - "Test economy in cards 8, 10, 11, 12 and 14: for each new test function or file, name the existing test that already covers the behavior, or the public-surface test that covers a symbol. Cards 8 and 14 each add a test beside existing tests in `cli_test.go`, `wiring_test.go` and `halt_test.go`."
  - "`Edit` cards that name symbols in their Intent: do the member glyphs match? Where an Intent changes a signature (for example `armFromSeed`, `resolvePersistentPreRun`, `stepCmd`, `wire`, `specFor`), is the re-sign arrow present? Struct fields and interface methods take no arrow."
  - "Card order across the loop chain: card 10 uses `KindInterrupted` (card 8), `WriteFailedStop` (card 5), `NewTraceID` (card 9) and `TreeRecord` (card 1). Cards 11 and 12 extend card 10's `LoopSpec`; card 14 uses the `shedrun` constructors (card 3) and the idle key (card 13). Does any card depend on something a later card creates?"
  - "Fidelity to the decision record: is every Decision (D1 to D10) and every Constraint carried by some card? Check D2's waiter and fswatch wakeups, D9's `ForkSubagents` and the `scribe:prose` drop, the orch stencil line, and D10's two trace-noise fixes. Does any card add scope the record does not license?"
  - "`## verify:` coverage: does every package a card targets run its tests, including `-tags integration`? Cards 1 and 2 touch platform-split code. Is the order the plan template states followed?"
---

Seed pass only: no round has been judged.
Each entry names a place to look and a question to settle, and none pre-states a verdict.
Mechanical checks, `DependsOn` completeness and `ImpactSummary` on non-`Edit`/`Delete` cards are skipped as the rubric directs.
