---
round: 3
focus:
  - "In `_lyx/discussion/decision-record.md`, D7's refusal for a twill fixer on an engine without fork support names a way forward, and D2's round fan pinning bullet says the fix half and a resumed half read the round's fans from the usage record, never from `loom.yaml`. Settle whether every step the refusal names can actually take effect for the round that was refused, given the fans were recorded when that round's review half started."
  - "In `_lyx/discussion/decision-record.md`, D7's `fix-fan` bullet hands the round's fixer fan to the segment's BurlerFix row, while D2 has the review half record both resolved fan names. Settle whether one carrier is described end to end for a shed run and for `lyx burler run`, given that `burlerReviewEntry` and `burlerFixEntry` in `internal/shedrecipe/entries_burler.go` are constructed separately and each reads only its own profile."
  - "In `_lyx/discussion/decision-record.md`, D2 makes the usage record the sole source of a round's fans for the fix half, a resumed half and the Bouncer. Settle whether D2 states when the names are written relative to the reviewer spawn, what a failed write does, and what a reader does when the names are absent, given that `internal/shedadapters/burlerreview.go` writes its part of the record only at acceptance and logs a failed write, and `mergeRoundUsage` in `internal/shedadapters/roundusage.go` replaces an undecodable record."
  - "In `_lyx/discussion/decision-record.md`, D4 adopts fork rules that cap each fork at 6 tool calls over an inherited pre-read, while D6 `affected-set` and D8 `fix-affected-set` each ask a fork for repository-wide searches. Settle whether the pre-read checklist or the lens text gives a fork what it needs within the cap, and whether Open risks names the cap against the Plan lenses as unbenched."
---

Entries are restricted to questions the artifacts must answer; none is drawn from the rubric's `Do not flag` list.
