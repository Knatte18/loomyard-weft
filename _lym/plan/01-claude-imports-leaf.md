# Card 1 — claude-imports-leaf

**Create:**
- `internal/claudeimports/doc.go`
- `plan:internal/claudeimports#Expanded` -> `type Expanded struct`
- `plan:internal/claudeimports#Expand` -> `func Expand(path string) (Expanded, error)`
- `plan:internal/claudeimports#maxImportHops` -> `const maxImportHops = 5`
- `plan:internal/claudeimports#TestExpand` -> `func TestExpand(t *testing.T)`
- `plan:internal/claudeimports#TestLeafInvariant_StandardLibraryOnly` -> `func TestLeafInvariant_StandardLibraryOnly(t *testing.T)`

**Extends:**
- `pattern/PATTERN-leaf-packages.md`

**Uses:**
- `internal/testkit/scankit#AssertImportAllowlist`

**Intent:**
Add `internal/claudeimports`, the one leaf that expands the `@` imports of a Claude instruction file by Claude Code's rule (D8).
`claudeengine` (card 7), heddle's render of the repository's CLAUDE.md into `AGENTS.override.md` (card 4) and the Codex instruction-file pointers (card 10) all call it,
so neither engine nor heddle restates the rule.

`Expand(path)` reads the file at `path` and returns `Expanded{Text, Files}`:
`Text` is the file's text with each import replaced by the expanded text of the file it names,
and `Files` lists the absolute, cleaned path of every file an import resolved to, in the order first met, each once.
The rule, from Claude Code's memory documentation:

- An import is an `@` that opens the line or follows whitespace, followed by a path running to the next whitespace.
- A relative path resolves against the directory of the importing file, and a leading `~/` against the user's home directory.
- An `@` inside an inline code span or a fenced code block is no import and stays as written.
- An import whose target is not a regular file stays as written, so a mention such as an e-mail address is untouched.
- Expansion recurses into an imported file's imports up to `maxImportHops` hops below `path`;
  an import past the last hop, or one naming a file already on the current import chain, stays as written.

A read error of `path` itself returns the error; a target that cannot be read stays as written.
The package imports the standard library alone.

`TestExpand` is a tier0 table test over files written under `t.TempDir()`, one row per behavior:
a relative import expands and lands in `Files`;
a nested import resolves against its own file's directory;
an import in a code span and one in a fenced block stay as written;
an import of a missing file stays as written;
a chain six hops deep stops expanding after the fifth hop;
an import cycle stops at the repeated file;
a `~/` import resolves under a home set by `t.Setenv`;
and an unreadable `path` returns an error.
`TestExpand` itself calls no `t.Parallel`, since `t.Setenv` panics under a parallel ancestor:
each row but the `~/` row runs as a parallel subtest, and the `~/` row runs as a non-parallel subtest.
`TestLeafInvariant_StandardLibraryOnly` is tier0 and calls `scankit.AssertImportAllowlist(t, "internal/claudeimports")`.

`PATTERN-leaf-packages.md` gains an `## internal/claudeimports` section: it imports only the standard library,
and it is the one expander of Claude instruction-file imports that `claudeengine`, `heddleengine` and `codexengine` share.

**ImpactSummary:** one new leaf package with no caller until cards 4, 7 and 10; `PATTERN-leaf-packages.md` gains one section; no existing symbol changes.
