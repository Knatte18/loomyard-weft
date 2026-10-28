```yaml
lens: fix-affected-set
outcome: reported
```

Task: through the fix-affected-set lens, check that each set-premise finding's fix covers every member of its set.

**List of findings left unchecked:** none.
I checked F1 and F4.
F2 and the parent's set decision name single rows, so neither has a set premise.
F3 is decision-class and outside the fixer's scope.

**F4, the cards that edit `validatePlanCmd`'s Long help.**
Cards 7, 8 and 9 are the only cards that list `internal/loomcli#loomCLI.validatePlanCmd` or edit its Long help.
Cards 1 and 4 edit `validateDiscussionCmd`'s Long help instead, which is a different command whose sentence the finding does not cover.
The fix covers all three members: card 7 rewrites the sentence, and cards 8 and 9 each add their check to it.

**F1, where the test-tier way-forward string appears.**
In the plan, the string appears only at card 7 line 43.
The only other place that should carry it is the `loom-plan-spec.md` row, and card 7's Docs paragraph now names "the way-forward string the finding carries".
No other card quotes the string.
Card 9's stencil sentence names `test-tier-unstated` but restates none of its rules, so it is unaffected.

no findings

Tool calls:
1. Grepped the plan cards for `validatePlanCmd`, Long help and help text, to rerun F4's search: my context showed only the help-text lines of cards 7, 8 and 9, not the other cards.
2. Grepped the plan cards for the way-forward string, `tier0` and `untagged`, to rerun F1's search: my context lacked the content of the other cards.
3. Read card 4 line 92 and grepped cards 1 to 6 for help edits, because the line was cut off in call 1 and I needed to rule out a `validatePlanCmd` help edit.
