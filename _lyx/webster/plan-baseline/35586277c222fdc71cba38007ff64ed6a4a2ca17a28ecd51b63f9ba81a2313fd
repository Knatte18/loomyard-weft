# Card 3 — vocabulary scan: drop the test-local records-name strip

**Delete:**
- `internal/lyxcwd#recordsRepositoryName`

**Edit:**
- `internal/lyxcwd#fabricVocabularyFailures`
- `internal/lyxcwd#TestFabricVocabularyFailures_Fixtures`

**Intent:** Carries D3.
In `internal/lyxcwd/enforcement_test.go`, the `recordsRepositoryName` constant and its doc comment go, and `fabricVocabularyFailures` drops the `if vocabScanRootFiles[rel]` block that runs `strings.ReplaceAll(text, recordsRepositoryName, "")` with its comment.
Root files keep their bare-token check through `fabricvocab.BareTokenInRootDoc`, which the same function already selects for them, so `PATTERN-fabric-vocabulary`'s bare-token scan still fails on every side token in a root file other than the records repository's name.
The host-phrase check also read the stripped text, but its scope excludes root files, so that path changes nothing.
`fabricVocabularyFailures`'s doc comment keeps naming the records repository's published name as the one `CLAUDE.md` and `README.md` may spell.
`strings` stays imported, since the file uses it elsewhere.

Two rows join the table of `TestFabricVocabularyFailures_Fixtures` in `internal/lyxcwd/vocabscan_test.go`, at `README.md`:
- a markdown link whose text is the records repository's published name and whose URL is its GitHub URL, the link the `named` string of the `root_doc_admits_records_repo_name_only` subtest in `enforcement_test.go` spells, then `\n`, passes (`wantFail` false), which fails if the root-file selection of `BareTokenInRootDoc` is lost now that no strip hides the name;
- the same text followed by ` Each task has a weft branch.\n` fails (`wantFail` true).
The scan over the current tree, which must report no failure after the change, stays pinned by the existing test that runs `fabricVocabularyFailures` over the module root.

**ImpactSummary:** Test-only change inside `internal/lyxcwd`: one constant with its one use removed; `fabricvocab` and its `recordsRepoNamePattern` are untouched, and the scan's verdict on `CLAUDE.md` and `README.md` is the same since every occurrence there sits between brackets, slashes or parentheses.
