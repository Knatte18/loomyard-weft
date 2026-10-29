---
round: 2
focus:
  - "Cards that add `Landing.PullRequest.*` func seams or nested `Deps` structs (cards 11, 13, 14): check whether `internal/battencli/wire_seams_test.go` and `internal/loomcli/wiring_seams_test.go`, their `intentionallyNil` maps and the `//lym:seam` markers are edited by a card at or before the first card that adds a seam, so no card's tier0 gate fails on a seam test."
  - "Card 13's `PRReview.wait` and card 14's session write: settle whether the record is re-read on each wake before the `Continue`, `ClosedForContinue`, `GatedHead` and `MailedHead` judgments, and whether every record write by the row goes through `pullrequest.UpdateRecord` so a concurrent `lym loom pr-continue` is not overwritten."
  - "Card 13's closed-pull-request verdict and card 15's `pullRequestLanding`: settle whether a closed pull request ends Done on `Continue` alone, and whether the clean-tree and head-equals-`GatedHead` checks of the open-with-`Continue` verdict apply on that path."
  - "Card 13's wake step 4: settle whether a record carrying `Continue` on a dirty tree or an ungated head can read `Status` on every wake, against the once-a-minute poll bound in the decision record."
  - "Card 8's `ResolvePullRequest`: settle how the origin host is judged to be github.com versus another host, and whether a `ParseOwnerRepo` refusal of a github.com-shaped origin under parent on can become a silent no."
  - "Cards 4, 6 and 11 and every doc, `doc.go` or pattern file they falsify: settle whether the new `pr-review` gate key, the shellguard and claudeengine doc text, the loomrecipe row sequence, the `pawl-recipe.yaml` header and the shedrecipe flag example are on a card's target list with the rewrite in its Intent."
---

Focus targets the plan as it stands after the round-1 fixes; the pause-step decision is settled through the parent and is not repeated here.
