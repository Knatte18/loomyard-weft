---
round: 1
focus:
  - "internal/boardengine registry and hold setters (machines, holds in board.json): does every write go through the one setter with a synchronous push, and does each D3 refusal fire on its path?"
  - "Every remaining recorded-machine-name comparison outside boardengine (hubgeom, beamcli, battencli, pairteardown, tendercli, adoptcli, loomcli, orchcli): does each go through Registry.ReadsAsHere or OwnedElsewhere, or is a raw name equality left?"
  - "internal/dobbyengine CatchUp and its loop wiring: is it run before any act rule answer after a sleep, a dobby start and a non-continue pass, and does a failed fetch or push leave the owed marker and the run's state correct?"
  - "ReadRun and the act rule (internal/runfsm Act, loom resume, tender resume): does the held refusal read the Bolt hold after a Bolt fetch, and does a hold keep queued tasks from being claimed?"
  - "Retirement of the owner pause caller and deletion of PauseOwnedElsewhere: assert-no-callers holds, and the upgrade's first read turns standing adopt entries into pause intents plus one Bolt hold without duplicating on a second read."
  - "internal/machinename and lym machine name (--forget-former, id keep or replace): does the identity write under the machine file's lock stay atomic, and does a forgotten name never read as here?"
  - "pattern/PATTERN-owner-machine.md: does the changed line land with the declared text, and do the PATTERN-cli-cobra and PATTERN-dobby-runs edits match what the cards ask for?"
  - "New tests flagged contained in the redundancy report (TestOwnerPreflight_TeardownResume, TestRun_ALaterArchiveFailureLogsAgainAfterTheOwnerLeaves, TestPlanPass, TestReadyToStart): compare assertions against the covering tests before filing."
---

Focus set from the decision record's scope, the plan's card index and the redundancy report.
