# Card 6 — seed-commit-signature

**Edit:**
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/fabricengine#BinaryLabel` -> `func BinaryLabel() string`
- `plan:internal/fabricengine#binaryLabel` -> `func binaryLabel(id buildvcs.Identity, channel, executable string) string`
- `plan:internal/fabricengine#SeedCommitMessage` -> `func SeedCommitMessage(subtree, label string) string`
- `plan:internal/fabricengine#SyncCommitMessage` -> `func SyncCommitMessage(subtree, label string) string`
- `plan:internal/fabricengine#IsSeedCommit` -> `func IsSeedCommit(message string, paths []string) bool`
- `internal/fabricengine/seedsignature_test.go`

**Uses:**
- `internal/buildvcs#Identity.Label`
- `internal/buildinfo#ChannelProduction`
- `internal/fabricengine#StencilsSubtreeRel`
- `internal/fabricengine#SpecsSubtreeRel`

**Intent:** D3 and D5's signature, declared once in `fabricengine` beside the subtree helpers in `stencilcommit.go`.
`binaryLabel` renders `(<label> <channel> <executable>)` from an identity's `Label()`, the channel string, `unstamped` when empty, and the executable path; `BinaryLabel` feeds it `buildvcs.Running()`, `buildinfo.Channel` and `os.Executable()`, `unknown` when that fails, and is the one label the pre-run, `lyx stencil sync` and `hubreconcile`'s hub-wide commit share.
`SeedCommitMessage` is `lyx: seed <subtree> <label>` and `SyncCommitMessage` is `lyx: sync <subtree> <label>`, the label already parenthesised.
`IsSeedCommit` is D5's recognizer: the message's first line is `lyx: seed <subtree>` followed by one space and a non-empty parenthesised label, and every path lies under `StencilsSubtreeRel()` or `SpecsSubtreeRel()`; a bare `lyx: seed <subtree>` from a pre-change binary, any `lyx: sync` commit, a hub-wide config commit and a commit touching a path outside the two subtrees all answer false.
The new untagged test file holds `TestSeedCommitSignature`, a table over `binaryLabel`'s forms and `IsSeedCommit`'s accept and refuse rows; nothing here spawns a process, and the budget row for `internal/fabricengine` rises by one.

**ImpactSummary:** Adds five package-level symbols with no callers until cards 7 to 10 and raises one budget row; no existing code path changes.
