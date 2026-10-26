```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fix change set (F1 to F4) against its surroundings.

No findings.

- F1: `pattern/PATTERN-refusal-way-forward.md:18` and `:23` now agree with `cmd/lyx/norecoverydeny_test.go:228`, which asserts `sed -i` is not denied. The "closed list" bullet no longer names `sed -i` or `perl -i`, and the computed-path `rm` and `systemctl` bullet is intact.
- F2: the `rows.go` comment no longer names `unquotedPrefixPattern`, and a grep finds no other reference to that symbol.
- F3: the test and its doc comment are back to the single "Edit or Write" anchor. The remaining `sed -i` mentions are intentional:
  - `internal/editdirective/doc.go:7` and `contracts/stencils/edit/edit-directive.md:6` state the scripted-edit rule.
  - `internal/shuttleengine/claudeengine/doc.go:222` records that no row denies scripted edits.
  - `internal/shellguard/rows_integration_test.go:209` is an allowed-everywhere row.
- F4: the `docs/research/codex-feasibility.md:14` row no longer lists `sed -i`. The other `docs/research` hits are dated reports, which the review exempts.

Out of scope, noted only: the burler fixer prompt's own "never `sed -i`" wording is generated text, not a repo file this change touches.

Tool calls:
1. Grep across `.go`, `.md` and `.yaml` files for `sed -i`, `perl -i`, `unquotedPrefixPattern` and `in-place`: my context had no repo-wide twin search.
2. Read `rows_integration_test.go:195-235`: to confirm the `sed_in_place_edit` hit is an allowed-everywhere row and not a stale deny.
