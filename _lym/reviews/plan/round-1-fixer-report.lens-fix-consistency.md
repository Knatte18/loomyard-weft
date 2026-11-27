```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with their sibling decisions, cards, rows and cross-references, and look for a change whose twin elsewhere was left unchanged.

1. LOW — Card 13's new `cmd/lym#testOnlyCodeAllowed` entry for `internal/fabriccore.Refusal.Error`, and the overview decision "the foundation's unreached loop functions take dead-code allowlist entries" ("Card 13 adds the entry … card 14 updates the allowlist's doc comment").
   - Summary: between cards 13 and 14, the guard holds an entry of the new temporary kind while its doc comment still admits only shared helpers, so the comment is false for one card.
   - Evidence: `cmd/lym/testonlycode_linux_test.go:41-46` reads "An entry is a reference or helper the tests of other packages share". Card 13 adds no change to that comment, and card 14's Intent makes the comment edit.
   - Note: the parent's decision entry assigns the comment update to card 14, so moving it would depart from that ruling. The gap needs either a mention in the fixer-report or the comment update moved to card 13.

2. LOW — Card 3's new **Edit** glyph `internal/fabricengine#TestFilterHubReserved`, which cuts its two `HubReservedNames()` loops.
   - Summary: card 3 now edits `junctionnames_test.go` explicitly, but nothing rewrites that file's header, which goes stale once card 3 moves or deletes the tests it names.
   - Evidence: `internal/fabricengine/junctionnames_test.go:1` reads "unit tests for junctionNames, filterHubReserved, BoardDir, HubPath and IsReservedHubName". Card 3 deletes `TestBoardDir` and `TestHubPath` and moves `TestIsReservedHubName` to the layout. Card 2 got the matching header line for `structuraldirs_test.go`; card 3 has none for this file.

Further tool calls: none; my context held every file the check needed.
