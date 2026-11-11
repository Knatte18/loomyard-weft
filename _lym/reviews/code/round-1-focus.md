---
round: 1
focus:
  - "Issue 97: the implementer and master stencil diffs. Does each rule make a fix after a pushed card commit land as a new commit, and drop no existing stencil rule?"
  - "Issue 101, `internal/webstercli/ownership.go` and `cli.go`: does a Master session with no live run lock or an ended run refuse to act? Does `record-batch` without the lock refuse rather than warn? Check each failure path."
  - "Issue 101, `internal/loomcli/decision.go`: does `lym loom decision add` skip or warn about a producer strand whose run is halted? Does a live, unhalted strand still get notified?"
  - "Issue 106, `internal/websterengine/verifygate.go` and `verifygatereport.go`: do the attempt count and the limit agree in the halt reason, including the zero-failure and exact-limit boundaries?"
  - "Issue 106: does the halt reason name the failing gate, target and output tail, say so when no failure is named, and carry a way-forward line? Does it follow the form of commit 1b6e3fca for darn?"
  - "Scope: the decision record offers batten's leased force-push of an amended card commit only as a fallback. Does the diff add anything the record does not license?"
  - "Tests: for the new tests in `decision_test.go`, `ownership_test.go` and `verifygate_test.go`, check that each pins a behavior the others do not and that every added behavior has coverage."
  - "Doc comments in the three touched `doc.go` files and the commit messages: do they match what the diff does?"
---

Round 1 seed. Three issues (97, 101, 106) across stencils, `webstercli`, `loomcli` and `websterengine`.
