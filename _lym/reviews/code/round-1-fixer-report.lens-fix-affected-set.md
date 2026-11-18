```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set lens — check the fix covers every member of the sets the findings name.

No findings.

Set-premise findings I searched, each covered:
- **F4 (the seam):** `CodeDiff` is declared (`recipe.go:295`) and refused when nil (`entries_burler.go:103`). It is wired in the one real Env builder (`wiring.go:1161`), defaulted in `envkit.go:552`, and listed in `nilLegal` (`envkit.go:142`). No other Env builder wired `CodeSubject`, so none needs `CodeDiff`. The three recipes with `stage_diff: true` plus the tapestry testdata fixture are served by `envkit`.
- **F4 (references):** no code, doc or spec still says `Subject.Patch` feeds the pieces. Remaining "patch" mentions are the Bouncer's own, which F4 left unchanged. `Profile.Diff` doc in `profile.go:66` names no source.
- **F2 (the 60,000 figure):** no `60,000` or `60_000` remains for the piece cap. `writeDiffPieces`, `packUnits` and `boundedUnits` callers are only `review.go:126` and the tests, and all use `pieceCap`.
- **F1 and F11 (templates):** `TestTemplate_FillsWithAllMarkers` and `TestTemplate_OptionalDirectives` are the only candidates named, and both carry keep markers. `burler-step-1-explore.md` already lists `diff_pieces` as an optional marker.

Unchecked: the F5–F10 test rows have no set premise, so I did not search them.

Calls:
1. Ran a repo grep for `CodeDiff`, `ReadCommittedDiff`, the 60,000 figure, the piece-function callers and `stage_diff`, to cover F4 and F2 in one pass, since my context lacked the whole-repo hits.
2. Ran a grep for Env literal builders, the template tests, and `diff_pieces` references, to find any Env builder other than `envkit` and `wiring.go`, and any other template test.
3. Ran a grep for stale "subject's patch" wording, and read `profile.go:55-70`, to check docs that still describe the old source.
