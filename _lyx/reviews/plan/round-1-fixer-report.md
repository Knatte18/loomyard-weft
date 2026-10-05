# Round 1 fixer report

## Changed

- F1: `00-overview.md` retitles the `git reset --keep` decision as a departure from D7 for operator confirmation, with A6's false premise stated and D7's verb rule marked as carried for every `start` and `pre-fix` site.
  Cards 12 and 15 each carry a one-line pointer to it.
- F2: card 15's D10 paragraph now states whole-token matching, the marker's bound (a direct prohibition only, a denied form offered later in the sentence still fails), and the self-tests for the offered-after-marker case and for `git reset --keep <sha>` and `git checkout <head> -- <path>` passing.
- F3: card 4's Intent names the compound-command matching risk and its fallback trigger.
  The smoke test now installs its own allow and deny rules in the hub's project settings, skips only without the `claude` binary, and the card says `verify:` compiles it and does not run it.

## Deferred

- F1's underlying choice, whether to keep `git reset --keep` or widen the reset verb's targets, is the operator's decision against D6 and A6.
  The plan keeps the narrower route and flags it; the approval flag in the overview is untouched.
