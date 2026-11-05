```yaml
lens: fix-consistency
outcome: reported
```

No findings (lens fix-consistency: whether the change set agrees with its sibling texts).

The fix changes when a plain promote gives recipe raddle: a discussed note that carries the sketch label now takes raddle even when its record file is missing. Three sibling texts say less than that rule, but each is still true.

- **`internal/boardengine/board.go:415` (the `PromoteTo` doc comment):** it says a discussed note "whose decision record is stored" takes raddle on an empty recipe. It is incomplete for the labelled case, because `store.go:765` was widened and this twin was not. The sentence is still true and the code is correct.
- **`internal/boardcli/cli.go:174-176` and `internal/boardcli/doc.go:79-80`:** both limit the promote rule to a Sketch or a note with a stored record. A Sketch is a note carrying the label, so the labelled case is already covered by "a Sketch". Neither text contradicts the code.
- **`internal/boardengine/labels.go:52`, `labels.go:67` and `sketch.go:92`:** they say the plain promote gives recipe raddle, and the fix makes that true for the missing-record case too.

Sketch-by-handle (`S1`) is untouched.

The new test row matches its siblings:
- It uses the shared `sketch` setup helper.
- It wants `wantRecord` false, which matches the record file being removed.
- Its wants match the other raddle rows.
- The `TestDiscussedPromotion` doc comment still describes the rows accurately.

Tool calls:
1. `grep` over `internal`, `cmd` and `docs` for the "gives raddle", "record file is missing" and `recordStored` wording, to find twin texts my context lacked.
2. `Read` of `board.go:408-423`, because the `PromoteTo` doc comment was not in my context.
3. `Read` of `cli.go:170-179`, because the CLI help text was not in my context.
