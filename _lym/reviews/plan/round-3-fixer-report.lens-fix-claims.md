```yaml
lens: fix-claims
outcome: reported
```

Checked every claim the fix added or rewrote in the changed text of cards 7, 8 and 9 against HEAD.

no findings

- The Long help sentence that cards 7, 8 and 9 now name matches `internal/loomcli/validate.go:181-182`, apart from its line wrap.
- It is the last sentence of the mode list, so naming it by how it starts identifies it.
- The support-only form in the new way-forward string ("the word `untagged` when the file holds support only (TestMain, helpers) and carries no build line") holds.
  `pattern/PATTERN-test-speed.md:17` says "an untagged test file holds support only, such as `TestMain` and helpers".
  `internal/loomshed/testmain_test.go` "carries no build tag, so it compiles into every tag set".
- The set-of-tags sentence and its two-lines-both-`tier1` row agree with the pass rule "exactly one tag" and with the record's added decision.
- The helper row's expected Detail, naming `tier0` and the own-line clause, follows from the card's own carry rule.
  `untagged` is not a tier tag, so `tier0` is the one tag the Detail names.

Tool calls:
- One Bash call read the head of `internal/loomshed/testmain_test.go`, the `untagged`/`TestMain` lines of `pattern/PATTERN-test-speed.md` and `internal/loomcli/validate.go:165-185`. My context held only the review's quotes of these and one grep line, not their text at HEAD.
