```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Fork task: for F1 and F2, rerun each finding's search to check the fix covers every member of its set.

- **F1** (sites reading `cardMentionRe`): the only use is `internal/planparser/renumber.go:683`, in `rewriteCardMentions`, and the hyphen skip sits there. `maskCardIDs` blanks card ids before matching, so the skip sees the masked text. The fixture's command example (`insert-card` + `first+2`) sits in the framing, which every `TestRenumber` row checks through the overview-outside-Card-Index comparison. In the remove-second and insert-second rows that card moves, so the test fails without the skip. `remove-card` is the same hyphen shape and takes the same code path, so it is covered without its own fixture line.
- **F2** (spec wording): the old sentence "A card id is never reported" appears nowhere else. The grep hit `internal/planparser/renumber_test.go:420` ("a card id is never read as a bare reference… which is reported") is accurate and doesn't contradict the new spec wording.

Calls made:
1. Grep of `*.go` and `*.md` for `cardMentionRe`, `card id is never` and `never reported`: to find any other consumer of the regex and any other copy of the F2 sentence, which my context didn't show.
