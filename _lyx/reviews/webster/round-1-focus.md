---
round: 1
focus:
  - "Cards 1–3 and 7 (`internal/planglyph`, `internal/planparser`, `internal/webstercli` dispatch): does `planglyph.ValidateDispatch` scope D1–D4 to the unbegun cards only, and do the whole-plan entry points still refuse a missing Delete target?"
  - "Card 3: is the already-deleted Delete target downgraded to an advisory only when at least one card is begun, and does the unbegun card's own mechanical check still run against that group's targets?"
  - "Card 7: does a `delete-not-done` caused by a later card name that card, and does `recover-batch` refuse to loop instead of retrying the same stop?"
  - "Cards 5 and 6 (`begin-batch`, `rebaseline --card`): are the archive of a report with no begin-batch record and the edit to a failed, dead or stuck batch's card correct in every batch state, with no state.json write skipped on an error path?"
  - "Card 4: is every trace of begin-batch's Master model assertion and injection removed, with no remaining reader of what it set?"
  - "Cards 9–11 (`gitrepo`, `fabricengine`, `webster reset`): does the leased remote update fail closed under the ancestry gate and the lease, and can reset never move the remote task branch past commits HEAD lacks?"
  - "Card 12 (Publish push-rejected halt): do the named remote tip, commit count and resume command match what `Fabric.RemoteOnlyCommits` returns, and does the resumed path reach the GitHub step?"
  - "Each card's own groups: did the type-specific mechanical check (`Rename`, `Delete`, per the plan spec's table) run against every group's targets, not only the first label's?"
  - "Test economy: does a new regression test duplicate a row or scenario the two test prunes left in `verbs_test.go` or the `wiring_*_integration_test.go` files, and does each added behavior have coverage that drives the real cobra command or `wire()` seam?"
  - "Doc comments and docs: do new or changed comments follow the target repository's constraints document, and did `doc.go`, `docs/overview.md` and `PATTERN.md` change in the same commit where a card changed observable CLI behavior?"
---

Focus is set from the plan overview and card index alone; no round has been judged yet.
Entries name where to look and the question to settle, none pre-states a verdict.
