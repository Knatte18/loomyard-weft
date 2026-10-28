```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: I checked the change set against sibling cards, D7 and cross-references.

- LOW: two of the changed sentences in card 7 word the own-line rule differently.
  - Changed text: in card 7's Intent, the way forward says "on a line that names no other test file". The new Docs card rule says "on a line that names no other test file of the card".
  - Summary: the way forward forbids naming any other test file on the line, including an existing one the card does not create. The spec card rule forbids only test files of the card. A writer who follows the spec could keep a line that names an existing test file with no tag, and the check passes it. So the stricter way-forward wording asks for more than the check needs.
  - Evidence: card 7's pass rule collects tags only "for each ref of a card's `Create` group … ending `_test.go`". Only another created file sharing the line merges tags.
  - Wording drift in the same change: the new Intent sentence says a shared line "gives each the tags of both", and the Docs spec row says "gives each the tags of all".
- I found no conflict with D7. D7's way forward is "add an Intent line naming the file and its `//go:build` line, such as `//go:build tier0`". The fix adds a clause to it and leaves D7's pass rule unchanged.
- No other plan card restates the way-forward text. Card 9's stencil sentence names `test-tier-unstated` but restates none of it, so nothing else needed the same change.
- The existing row "`tier0 tier1` on the naming line fails, its Detail naming both tags and the join-or-keep-one clause" still matches the reworded "join them with || only for … else keep one".
- Card 9's move of `validatePlanCmd` to Edit agrees with cards 7 and 8, which list it under Edit. Card 9's Docs line, "`validatePlanCmd`'s Long help names `doc-go-unlisted`", is unchanged and still fits.

Tool calls:
1. grep of `_lym` for the way-forward text and `test-tier-unstated`, to find siblings outside cards 7 and 9 that my context lacked.
2. grep to locate the decision record holding D7, which my context lacked.
3. awk print of D7 from `_lym/discussion/decision-record.md`, to check the changed way forward against its decision.
