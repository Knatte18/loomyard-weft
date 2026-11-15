---
round: 2
focus:
  - "Card 5 (`pairteardown` ReadRun): confirm `internal/pairteardown` imports neither `battenrecipe` nor anything reaching `hubforge` in tests; `internal/pairteardown/followup.go` keeps `landRow` and `worktreeTeardownRow` local for this reason. Settle whether the card builds its own gate."
  - "Card 11 (`dobbyengine`): `internal/dobbyengine/seam_enforcement_test.go` holds a closed import allowlist; settle that every import the card adds is listed on the card and that `drivable`, `RunView` and `DobbyRuns` carry no clause the act rule makes unreachable."
  - "Cards 5 and 11: settle whether any read the dobby pass makes per listed run, per tick, runs `git ls-remote` or another network call, against card 4's statement that the FSM's origin reads make none."
  - "Cards 7 and 8: settle whether a cancel is visible to every reader (start guard, pre-drive check, DobbyRuns) from the moment its lock is taken until its intent is committed."
  - "Card 3 and the cards that word its refusals: settle that each refusal's way-forward verb lifts the cause that produced it, for a pause from the record's intent with no local pause entry, and for a landed slug whose pair still stands."
  - "Overview Shared Decisions: settle that every aside the plan sets (including the shed-pause aside) is licensed by the decision record or an `Added after Discussion` entry."
  - "Every fix applied since the first review is new text: check the changed cards against the `Do not flag` list, card order, and the PATTERN entries `PATTERN-test-only-code`, `PATTERN-leaf-packages` and `PATTERN-fabric-boundary`."
---

Entries name where to look and the question to settle; none pre-states a verdict.
