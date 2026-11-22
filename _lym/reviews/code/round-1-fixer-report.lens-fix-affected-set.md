```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Rerun of F1's search (every `--text`, `RenderCompact`, `prime owner:` and `sketches:` use in `*_test.go` outside `.lym` and `_lym`):
- **Fixed sites:** `internal/boardcli/verbs_test.go` — `stepListAndFindText` and `stepListShowsASketch` no longer pin column padding or the `sketches:` line.
- **Other boardcli `--text` sites:** all hold.
  - `verbs_test.go` lines 397 and 403 check only the `P0 ` prefix and line count. The review said to leave them.
  - `verbs_test.go` line 478 checks only the `prime owner: neve\n` prefix.
  - `help_test.go` lines 121 and 127 are help-text substrings.
- **Out of the board set:**
  - `internal/orchengine`'s `RenderCompactFocus` is a different renderer.
  - `internal/quarrycli/verbs_test.go` `glyphs --text` tests select lines by id field and check emptiness; they are not layout goldens.
- **Leftovers:** none. No exact-string compare of the compact listing remains.

Further tool calls (2):
1. A grep over `*_test.go` for the `--text` and compact-listing markers, to rerun F1's own search. My context held only `verbs_test.go` lines 300–450.
2. A read of `internal/quarrycli/verbs_test.go` lines 245–285, to see whether its `glyphs --text` tests pin layout. My context lacked it.
