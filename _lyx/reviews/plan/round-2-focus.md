---
round: 2
focus:
  - "Card 8 against internal/landingshed/publish.go: does the card now cover every step-reachable GitHub and push failure arm in Publish, including checkApproval's FindPullRequest query on the approval path, and does each arm it edits call cancelErr before returning the classified hard error so that a cancelled step still returns the cancellation?"
  - "Card 8 arm coverage beyond Publish.Call: does any other landingshed or loom-recipe producer on a step path make a remote git or GitHub call whose failure still becomes Stuck, with neither card 8 nor its stated out-of-scope list covering it?"
---

Round 1's BLOCKING finding is the missing approval-path arm in card 8.
The second entry checks whether any other such arm was missed.
