---
round: 3
focus:
  - "Test-only-code guard timing: `internal/gatecli/doc.go` (Guard step) says the gate verb runs every `//lym:guard` test after any `--card` scope, and `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` in `cmd/lym/testonlycode_linux_test.go` carries `//lym:guard`. For each card that adds a function with no caller reachable from a main at that card's tree, or deletes the only caller of one, does the plan list the `cmd/lym#testOnlyCodeAllowed` allowlist entry with a Why naming the card that reaches it, and does a later card remove exactly the entries it makes reachable? Check the overview's Shared Decision on when the guard first runs against the same code."
  - "FixHold and ClearHold in `_lym/plan/02-board-adopt-write.md`: after the push, is success reported only when the re-read of the local Bolt matches what the setter wrote, and does a test row reach the case where origin's replay dropped the change without another machine's hold standing?"
  - "`runHoldCheck` in `_lym/plan/08-adopt-check-ends-on-hold.md`: is the hold re-read before the first green end removes the fix task, so an adopter whose hold another machine took neither cancels nor removes a task that machine owns? Does a test row reach that order?"
  - "`runForeground` step 2 and `movedRun` in `_lym/plan/09-adopt-foreground.md`: for a take-hold back from a machine that took the fix task's run over, does the fix-task clause defer to a takeover entry naming another machine in the origin record, and does a test row reach that input?"
  - "`TakeOverRun` in `_lym/plan/03-fabric-takeover-writer.md`: which teardown lock mode does each archive and push hold, and does any step take a lock the same call already holds, as `ArchiveOwnerTips` does through `archiveOwnerPair`? Does each prescribed error branch, the aborted rebase included, have a named input and test row that reaches it?"
  - "`holdCheckDeps.Green`, `greenDeps` and `hubGreenDeps` in card 8: does every field a seam's deps struct gains have a caller that sets it, and is every function whose result type changes listed as an Edit target?"
  - "Card 9's Edit and Delete lists against every file importing `internal/adoptrecord`, test files in the `adoptcli` external test package included: after the package is deleted, does any file still import it?"
---

Round 3 targets the keys still open after round 2, phrased as questions about the plan's text.
