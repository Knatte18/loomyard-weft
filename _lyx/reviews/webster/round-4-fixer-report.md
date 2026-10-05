# Round 4 fixer report

## Fixed

- F1: `readEscalation` now wraps every malformed-record case in the new `shedadapters.ErrEscalationMalformed`, naming the file.
  `circlingVerb` maps it to its own refusal with a `way forward:` clause.
  Added a loom spec row, and test rows in `circling_test.go` and `escalation_test.go`.
- F2: `decision add` status-read and other append failures end with a `way forward:` retry clause; test rows assert both.
- F3: the escalation-unreadable Reason carries the `way forward:` label; its test asserts it and the spec cell quotes the new wording.
- F4: the config review-key row's cell now quotes the message's wording.

## Deferred

Nothing deferred.
