# Card 2 — vocabulary-name-check

**Create:**
- `internal/planparser/vocabname.go`
- `internal/planparser/vocabname_test.go`
- `plan:internal/planparser#fabricVocabularyNameCheck` -> `const fabricVocabularyNameCheck = "fabric-vocabulary-name"`
- `plan:internal/planparser#CheckFabricVocabularyName` -> `func CheckFabricVocabularyName(plan *Plan) []ValidationError`
- `plan:internal/planparser#hasTestPrefix` -> `func hasTestPrefix(name string) bool`
- `plan:internal/planparser_test#TestCheckFabricVocabularyName` -> `func TestCheckFabricVocabularyName(t *testing.T)`

**Edit:**
- `internal/planparser/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planparser#HandleUnit`
- `internal/planparser#HandleMember`
- `internal/planparser#HandleIdentifier`
- `internal/planparser#IsHandleRef`
- `internal/planparser#cardID`
- `plan:internal/fabricvocab#BareToken`
- `plan:internal/fabricvocab#SkipsBareCheck`
- `plan:internal/fabricvocab#IsDocRoot`
- `plan:internal/fabricvocab#IsHostGeometryIdentifier`
- `plan:internal/fabricvocab#SideTokens`

**Intent:**
D9: a `planparser` function that refuses a name a plan declares which the fabric-vocabulary scan would refuse once it lands.
It sits outside `ValidateFormat` and `Validate`, like `CheckCardFabricReference`; card 3 appends it in the plan index.

`CheckFabricVocabularyName(plan)` reads, per card, the handle of every `Card.Declarations` entry (the `Create` handles) and the `New` side of every `Card.Pairs` entry that `IsHandleRef` reports as a handle (a `Rename` pair's new name).
A handle whose unit or member does not split (`HandleUnit` or `HandleMember` reports false, or either half is empty) is skipped, since `handle-malformed` or `handle-name-failed` already reports it; a `Create` bullet that failed the declaration grammar never reaches `Declarations`, so it raises no second finding.
For each name, with `unit` the handle's unit half, `first` its first path segment, `member` the whole member half and `last` its last dot-segment (`HandleIdentifier`):
- the bare-token rule fails when `fabricvocab.BareToken(member)` holds, `fabricvocab.SkipsBareCheck(unit)` does not (so an owner-set package passes and `internal/configsync` still fails, its identifier rule), and either `first` is `internal` or `fabricvocab.IsDocRoot(first)` holds, or `hasTestPrefix(last)` holds;
- the host-geometry rule fails when `hasTestPrefix(last)` does not hold, `fabricvocab.IsHostGeometryIdentifier(last)` holds, and `first` is `internal` or `cmd`.
`hasTestPrefix` reports a `Test`, `Benchmark`, `Fuzz` or `Example` prefix.
The bare-token rule reads the whole member, so a `Type.Method` whose type segment alone carries a side token fails; the other two rules read `last`.
A name failing both rules gets one finding.
The scan's package-name carve-out does not apply: a declared member is never that package name, so the check calls `BareToken`, not `BareTokenIn`.
Each finding is `ValidationError{Check: fabricVocabularyNameCheck, Card: cardID(card), Ref: <the handle>}` with a `Detail` naming the card, the member and `PATTERN-fabric-vocabulary`, the rule it breaks (a bare fabric side token, or a retired host-geometry identifier), and the way forward: rename the member.
The function runs under any `language:`, reads plan text only and only refuses.
No string literal in the new production file spells a side token: the detail says "a bare fabric side token", never the token.

`TestCheckFabricVocabularyName` (external package `planparser_test`, beside `validate_test.go`'s `validCard`) is one table over in-memory plans, each row a card carrying `Declarations` or `Pairs`, asserting the exact count of `fabric-vocabulary-name` findings and, for a failing row, the finding's `Card` and `Ref`.
Side-token names are built from `fabricvocab.SideTokens()`, never spelled, since the test file sits outside the owner set; host-geometry rows spell `hostBranch`, which the host rule leaves to non-test files.
Rows that refuse (the record's acceptance list): a `Create` handle `Test<Side>…` in a package outside `internal/`, such as `tools/x`; a member with a side token and no test prefix under `internal/`; the same under `cmd/`; a member with a side token in `internal/configsync`; a `Rename` pair whose `New` handle carries a side token; a host-geometry identifier under `internal/`, and as a method's last segment `Type.hostBranch`; a host-geometry identifier under `cmd/`; a `Type.Method` under `internal/` whose type segment alone carries the side token.
Rows that pass: a side token in an owner-set package other than configsync, such as `internal/fabricengine`; a member without a test prefix outside `internal/` and the doc roots, such as `tools/x`; a host-geometry identifier under `tools/`; a name with no token; a declaration whose handle carries no `#`, which yields no finding.
One more row pins one finding for a name failing both rules.

`internal/planparser/doc.go`'s sentence naming the functions outside `ValidateFormat` and `Validate` adds `CheckFabricVocabularyName`, which needs no told input but is appended by the plan index beside `CheckCardFabricReference`.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/planparser` by one for `TestCheckFabricVocabularyName`.

**ImpactSummary:** doc and budget lines only; the new check has no caller until card 3.

**Commit:** `2: Add the fabric-vocabulary-name plan check`
