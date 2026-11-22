```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The change set agrees with its surroundings.
- The `stepListAndFindText` rewrite and `textListingRows` match the kept checks at lines 403 and 478.
- `stepListShowsASketch`'s doc comment now matches its body.
- `text.go:25` still documents the `sketches:` line, and no test pins it any more, which is the intent of F1.
- No other test pins a column-padded `--text` layout. Hits in `help_test.go` and `quarrycli` are unrelated.

Tool calls:
1. grep for `sketches:`, `prime owner: ` and `textListingRows` in `*.go`: to find twins of the changed text outside the context I hold.
2. grep for `"--text"` in `*_test.go`: to find other tests pinning the layout, which my context lacked.
