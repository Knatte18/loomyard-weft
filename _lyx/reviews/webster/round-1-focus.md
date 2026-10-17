---
round: 1
focus:
  - "internal/creelengine/verbs.go Store.Log: does the id clause keep an entry whose Unread holds the id while From/To/Since still apply to every entry, and does a zero filter.ID still filter nothing?"
  - "internal/creelcli/cli.go writeRefusal and runVerb: is the stderr fallback correct for every combination of done, msg already prefixed with `creel: `, and a failed or partial success write? Does the lost-answer line bypass writeRefusal so no error envelope follows?"
  - "internal/creelcli/cli.go: does each verb body set `done` exactly as card 3 lists (post incl. --reply and returned-undelivered, inject all results, forward/hold/release, interrupt only when Sent), and leave it empty for inbox, log, read, no-running-turn and error-beside-result paths?"
  - "internal/creelcli/cli_test.go: do the new rows cover each behavior the card names without duplicating existing subtests (test economy), and is the changed `log --id prints a carried entry's kind` fixture (DormantStrand beta) still sound?"
  - "doc.go files and tools/sandbox/SANDBOX-CREEL-SUITE.md, and the logCmd help strings: does each card's D5 text land, and do doc comments of Store.Log, result, writeRefusal and runVerb state the new behavior?"
---

Three cards, three commits; the range is the branch's own commits over its parent.
