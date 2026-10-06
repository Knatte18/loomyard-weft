---
round: 3
focus:
  - "Comment line breaks in the diff's new or changed Go comments under internal/shuttleengine (including claudeengine/skillload.go and claudeengine/doc.go), internal/orchengine (doc.go, watcher_test.go) and internal/loomcli (integration_driverbootstrap_test.go, smoke_driverstrand_test.go): does any line keep two independent clauses, joined by a comma before and/so/which with its own subject and verb, on one line? A comma joining a list or a compound predicate does not count."
  - "Retry-step timeout in internal/orchengine: the watcher's reload sequence skips only the retry list with cause timeout when the retry turn passes the handoff timeout, and types no second retry. Does a test in watcher_test.go reach that path, so an implementation that skips the full skill list or retries again fails it?"
  - "Step constants in internal/orchengine tests: does any assertion compare ReloadStep with a literal where ReloadStepSkills or another named constant exists?"
---

The focus follows the open work only; every other part of the diff stays under ordinary review.
