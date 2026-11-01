---
verdict: APPROVED
---

## Review

The diff changes `PATTERN.md` and two `pattern/` background files, and it makes the change the amended board entry asks for.

- **Coverage.** Each of the six invariants lands as exactly one entry, with its rule intact. These are `doc.go` as documentation, fabric isolation, no hard-coded language, engine or branch, named constants, shed shared machinery, and lym's instructions kept in code.
- **Format.** The new `## Code` section and the entries added to existing sections follow the file's entry format. The two entries with a background file link to it. The three without one carry their whole rule in the line, which the record allows.
- **`(test)` marker.** No new entry carries `(test)`, and none claims a test, so no test is missing.
- **Accuracy of the background files.** Fabric is `internal/fabricengine` and `internal/fabriccli`. Its non-test imports include board, vscode, heddle, reed and webster packages, which matches the debt claim. The plan parser defaults an absent language to `go` at `internal/planparser/parse.go:146`. The shuttle provider seam is the existing `PATTERN-shuttle-provider-seam`.
- **Neighbouring entries.** `PATTERN-lym-instructions-in-code` governs what goes into heddle. `PATTERN-heddle-instructions` governs how heddle's store is rendered, and `PATTERN-stencil-ownership` governs how stencils are read. The new entry neither contradicts nor duplicates either.
- **Commit.** The message claims only what the diff does. No `Pattern-Departure:` trailer appears, and none is needed. The commit's attribution trailer names a different model than this session's reminder does. That is not a defect of the diff.

## Rejected

- Commit trailer `Co-Authored-By: Claude Opus 5.5`: raised in a fork's notes only, and the rubric lists no rule on trailers.
