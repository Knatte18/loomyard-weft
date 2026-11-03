```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F2 change set for twins left inconsistent.

No findings.

- The `cardMentionRe` doc comment (renumber.go:624) still reads as a plain match description. The hyphen exemption sits in the `rewriteCardMentions` doc, which is where the skip happens, so the two agree.
- The spec at `loom-plan-spec.md:225-227` now agrees with the code on all three points. Card ids are rewritten and reported under `rewritten_refs`; only a bare reference with no faithful rewrite is reported as a prose mention; "never needs a hand fix" holds. The spec does not mention the hyphen exemption. A `card N` glued to a hyphen is not a reference at all, so the spec's "such as `card N`" still holds, and no spec edit is required.
- I did not read `internal/loomcli/renumber.go:229` and the lines around it, the agent-facing help text for the verbs. A grep showed it says "a bare reference", and I saw no "never reported" wording elsewhere in the repo. I did not check whether that help text needs the hyphen note.
- The test fixture comment (renumber_test.go:35) matches the framing change. The new framing paragraph sits on its own line, so the prose-map keys (`Covers cards %d–%d.`) still match whole lines. The `first+2` reference moves in the insert-at-offset-1, insert-at-offset-2 and remove rows, so the overview comparison fails if the hyphen skip is removed.

Tool calls:
1. grep for `cardMentionRe`, "never reported" and "bare reference" in `.go` and `.md` files — to find twins and stale wording outside the files my context shows.
