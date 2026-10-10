---
round: 4
focus:
  - "PATTERN.md: after the commits that tightened entries to fit the merged overview, is the file at or under the 16384-byte cap declared as MaxOverviewBytes in internal/pattern/check.go, and does lyx gate test ./internal/pattern pass, including the 320-character entry-line cap, the [background] link resolution and the orphan-background check?"
  - "PATTERN.md: does every entry the tightening commits touched still state both its trigger (when it applies) and its requirement, with no rule dropped, in particular PATTERN-push-both-sides, PATTERN-shuttle-stop, PATTERN-parent-directive, PATTERN-review-round and PATTERN-batcher-registry?"
---

Round 4 checks the fix for the size-cap failure.
