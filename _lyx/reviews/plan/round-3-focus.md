---
round: 3
focus:
  - "Card 17 (`_lyx/plan/17-shuttle-shell-only-waiting.md`) changes `expiredTurnEnd` so an ungated run with an output file missing, whose waiting list holds only non-awaited shells all outstanding for `ShellWaitBound`, returns the held turn end. Settle whether every existing test in `internal/shuttleengine` and `internal/websterengine` that runs that state, and every comment or doc that states the old shell-wait rule, is a target of the card with its new expectation stated."
  - "Card 11 (`_lyx/plan/11-fabric-remove-reconcile.md`) makes `removeVerdict` refuse a `stopped` entry with the entry's way forward. Settle for each stopping branch state a run-scope reconcile can produce, including a branch gone from origin, whether the refusal's way forward is a step other than re-running `lyx fabric remove`, and whether every prescribed refusal has a named input that reaches it."
  - "Card 2 (`_lyx/plan/02-runproc-owner-record.md`) makes every `End` signal follow a fresh start-time comparison and a group signal conditional on a live member of the group. Settle whether each identity check, in both the SIGTERM and the SIGKILL phase, has a named `TestEnd` row that reaches it."
  - "Card 10 (`_lyx/plan/10-teardown-skip-readings.md`) defines `runLockHolderPasses` and `runLockRefusal`. Settle whether every input is read by its rule, whether the paused pass is stated for a known and an unknown holder, and whether each refusal's way forward is a step that can succeed for the state that produces it."
---

Focus targets the cards the round-2 review changed or found wanting, stated as questions to settle.
