---
round: 1
focus:
  - "internal/gitrepo/push.go, pushdestination.go: does the transfer run outside the remote-refs lock while the tracking ref and upstream are written under it afterwards, with the compare-and-swap `update-ref` correct when the tracking ref moved during the transfer?"
  - "internal/gitrepo/push.go: every failure path between the unlocked transfer and the locked ref write (rejected push, absent remote ref, lost compare-and-swap, retry) leaves the tracking ref and upstream consistent with what the remote holds."
  - "PATTERN-remote-refs-lock says a judge of origin holds the lock to its verdict: check every judge of origin (fabricengine/originreconcile.go, archive.go and other callers of gitrepo push) still holds the lock where a push may now land outside it, and that pattern/PATTERN-remote-refs-lock.md states the new rule."
  - "pattern/ and PATTERN.md diff: the pattern text changed with the code in the same diff, as the repository's own rules require."
  - "internal/gitrepo/remoterefslock_integration_test.go and push_test.go: do the new tests cover the behaviors the diff adds (transfer outside the lock, compare-and-swap loss), without duplicating existing coverage?"
  - "Commit messages in the range: any `Pattern-Departure:` trailer names a real entry that forbids the way taken."
---

Seed focus from the board entry's Idea: move the push transfer outside the weft remote-refs lock, and check every judge of origin before it may land outside.
The record has no Scope, Decisions or Acceptance criteria, so the Goal is the measuring stick.
