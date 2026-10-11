---
format: 5
approved: true
language: go
---

# Plan: burler-split-budget

A review segment's Burler round becomes two Shed rows, a review row then a fix row, each one fresh session on a one-seat seats table; the fixer spawns only for a review with findings, or for a clean review whose row gates fail, and orients on the review and its targets alone.
Each loom perch gets its own bounce budget and an auto-accept flag in `loom.yaml`, so a spent budget can settle the round as accepted instead of escalating to the parent.
The engine halves and the two producers land beside today's one-row round first (cards 2 and 3), card 4 switches both recipes to the split rows, card 5 moves `lyx burler run`, card 6 updates the prose naming the old rows, and cards 7 to 9 add the auto-accept settle and the per-perch keys.
Card 10 deletes the retired round, its seat channel, its ready marker and the old row names last, after every card that edits code naming them.

## Card Index

1 — review-carries-fix — every finding carries its `fix`, and the current round's review gate checks it strictly
2 — burler-halves-engine — `burlerengine` gains a review half and a fix half, each one session, with the lean fixer prompt
3 — burler-halves-producers — a review row and a fix row producer over the halves, with the round's acceptance record
4 — split-burler-rows — both recipes split every Burler row into a review row and a fix row, wired end to end
5 — burler-run-halves — `lyx burler run` runs the reviewer, then the fixer only when the review has findings
6 — row-name-prose — comments, specs and background files name the new rows
7 — bouncer-auto-accept — a Bouncer told `AutoAccept` settles a spent budget as an accept
8 — perch-budget-keys — `loom.yaml` gains per-perch budget and auto-accept keys
9 — perch-budget-wiring — each perch's rows carry its own budget and its Bouncer its flag
10 — retire-burler-round — delete the one-row Burler round producer, the two-seat engine round, its stencils, the ready marker and the old row-name constants

## Shared Decisions

### Decision: additive-then-retire

The new halves, producers and stencils land beside the old round under their final names, the switch moves every caller in one card, and the old symbols are deleted in the last card, once nothing calls them and no later card edits code naming them.
No card leaves a shim behind: the old round stays as it is until its deletion card, apart from `reviewReady` sharing card 2's `acceptReview` and card 4 dropping the old seam's compile-time assertions.

### Decision: acceptance-record

The round's acceptance record is `round-<n>-acceptance.md` in the segment's run directory, YAML frontmatter holding the round and the hex SHA-256 of the accepted review.
Only the review row writes it, only after the review passed `ParseCurrentReview`, and every archive of a whole round moves it with the review and the fixer report.

### Decision: changed-review-recovery

D4 makes a review changed after acceptance a hard error that archives the review, the fixer report and the acceptance record "so a resume re-reviews the round".
Shed resumes a failed run at the failing row, which is now the fix row, and only the review row reviews, so the fix row's next call meets the missing-acceptance refusal rather than a fresh review.
The plan keeps D4's hard error and archive set and gives that error the refusal's way forward, a goto to the perch's review row and then a resume, so the next review row call re-reviews the round; no automatic route back is added.

### Decision: row-keyed-perch-settings

The per-perch budget and flag reach `shedrecipe.Env` keyed by each review segment's Bouncer row name, as `RowBouncerModels` already is; 0 and absent mean the run-wide `review_max_bounces`, and the recipe builder resolves the fallback, so the engine and the routing projection apply one value.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
