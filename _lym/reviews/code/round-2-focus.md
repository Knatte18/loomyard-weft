---
round: 2
focus:
  - "internal/fabricengine/clone.go, ignoredWork: check the set of skipped ignored paths against every writer of a never-tracked file in the prime code store, the records store and the Bolt checkout, in both directions. Does any file lym writes still count as work, and does any skip hide a file the operator wrote? Check that the doc comment, internal/fabricengine/doc.go, the tests and the change description state the same rule."
  - "internal/fabricengine/clone.go, hubWorkOriginLacks and storeWorkOriginLacks: can --reset still remove commits, uncommitted changes or ignored files without naming them in any of the three stores, including a detached HEAD or paused rebase in the Bolt checkout and a store whose repository cannot be stat'ed or read?"
  - "internal/fabricengine/clone.go, checkResetTarget: does every refusal and read error it can return name a way forward that works from the state that produced it, and does a test reach it?"
  - "internal/fabricengine/clone.go, RefuseHubWithoutBolt: is every clone command it names accepted by the reset check for a used hub as well as a half-made one, and does it say what work that command loses?"
  - "internal/fabriccli: is the --discard-work flag's path from the cobra clone command into CloneOptions.DiscardWork held by a test?"
  - "tools/sandbox/SANDBOX-CORE-SUITE.md and the other documents, help texts and sandbox suites that mention clone --reset: does each match when a plain --reset is refused?"
---
