```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each finding's set search and check the fix covers every member.

- **F2 (LOW)** — filled-comma sites. Covered. `fillPromptPlaceholders` fills the ask in only two production sites, `buttonrow.go:117` and `buttonclick.go:118`, and the fix guards both.
- **F5 (LOW)** — sibling descriptions of the pre-ask entry. Three stale members remain, listed below.
  - `internal/reedengine/buttons_test.go:1`: "the placeholder filling of their run and confirm text" omits ask text.
  - `internal/reedengine/buttonclick.go:56`: "the run and confirm text and the confirmed line are quoted in" is stale for the ask path, since the answered line is also quoted in that dialect.
  - `internal/reedengine/bar.go:11` and `follow.go:2`: both list the decision-bearing options as `@lym_button_spec` and `@lym_button_dir` only, while the fix added `@lym_button_answer` to the `doc.go` list.
  - The `bar.go` and `follow.go` lines are mild; the `buttons_test.go` and `buttonclick.go` ones are not.
- **F6 (NIT)** — `cli_test.go:75` was updated. The test table (`cli_test.go:98`) has only a "button confirmed without hash" row and no `--answered` without `--hash` row. This is a coverage gap in an existing set, not a stale comment, and the review did not ask for it. The comment now claims coverage that the table lacks. I rate this LOW.

Unchecked: F1 (a single new test, no set), F3 (a single row) and F4 (the review named the two members, `{strand}` and `{beam}`).

Further tool calls (3 of 6):
1. `grep` for stale "confirmed without" and "run and confirm text" phrasings: my context lacked the other files.
2. `grep` for `fillPromptPlaceholders` and `.Ask` in non-test code: to enumerate production fill sites.
3. `grep` for "decision-bearing" and `@lym_button_dir` outside `doc.go`: to find sibling option lists.
