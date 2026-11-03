---
round: 1
focus:
  - "internal/planparser/renumber.go `mapsFaithfully` and `rewriteCardMentions`: for InsertCard and RemoveCard, does every bare reference form `cardMentionRe` matches (single, list with commas and `and`, ranges with hyphen, en dash and em dash) map to the right new number, and is a range or the removed card kept and reported exactly when the rewrite would change its meaning? Check the boundary cases: a range ending at the insert position, a range starting at it, and a reference to a card that does not move."
  - "internal/planparser/renumber.go `applyRenumber`: references are now rewritten in the same pass as the card-id rewrite, against the numbering before the call. Settle whether a card file whose only change is a bare reference is still written, journaled and restorable (the `len(refs) == 0` skip for unmoved cards), and whether `ProseMentions` still covers the overview and every card file, with heading lines and Card Index lines skipped as before."
  - "internal/planparser/renumber.go `cardIndexLines` and `maskCardIDs`: does the overview skip test select the same lines the removed `proseMentions` skipped, and is a card id such as `4-slug` never read or rewritten as a bare `card 4` reference?"
  - "internal/planglyph/index.go `glyphlessFile`: it matches the toc entry by `path.Base(file)` only. Settle whether TOC of one repository-relative path can return another file's entry, and whether every failure path (toc error, lossy, error entry, nil Symbols, missing file) leaves the original refusal in force. Check that ValidateFormat and ValidateRework pass the right root."
  - "internal/planparser/edittestfile.go `CheckEditTestFile`: the new `glyphless` argument is called only for a path the check would otherwise refuse. Settle whether every caller and the doc comment agree, and whether a nil `glyphless` is possible from any caller."
  - "Scope against the decision record's Goal: issue #4 (edit-test-file accepts a test file with no toc symbols) and issue #5 (insert-card rewrites bare card N references or reports each left one). Settle whether remove-card's change is licensed by the Goal, and whether the contracts/specs and contracts/stencils edits and the `doc.go` edits state what the code now does."
  - "Test economy in renumber_test.go, planglyph/index_test.go, edittestfile_test.go and loomcli/renumber_test.go: does each new test cover behavior no existing test covers, and does any behavior the diff adds (range kept, removed card kept, glyphless admit, quarry failure falling back to refusal) lack coverage?"
  - "Commit message and `_lym/landing/summary.md` claims against the diff, including the claimed `rewritten_refs` and `prose_mentions` report fields in internal/loomcli."
---

Seed focus for round 1 over the two commits in the review range: 68567859 (issue #4) and 17fe6785 (issue #5).
The decision record holds the board entry alone, with no plan, so the amended Goal is the measuring stick.
