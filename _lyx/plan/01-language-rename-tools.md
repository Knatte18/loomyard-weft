# Card 1 — language-rename-tools

**Create:**
- `plan:internal/languageconfig#renameToolsKey` -> `const renameToolsKey = "rename_tools"`

**Edit:**
- `internal/languageconfig#config`
- `internal/languageconfig#Language`
- `internal/languageconfig#ConfigSpec`
- `internal/languageconfig#Load`
- `internal/languageconfig#VerifyTiersValue`
- `internal/languageconfig#ConfigTemplate`
- `internal/languageconfig/template.yaml`
- `internal/languageconfig/doc.go`
- `internal/languageconfig_test#TestLoad_ResolvesSkills`
- `internal/languageconfig_test#TestLoad_InvalidFailsNamingKey`
- `contracts/specs/refusal-spec.md`

**Intent:**
D1.1 and D1.2: `language.yaml` gains a third key, `rename_tools`, an ordered list of strings, each a rule naming the hub's structure-aware rename tool for one language and how to run it.

- `config` gains a `RenameTools yaml.Node` field tagged `rename_tools`; `Language` gains `RenameTools []string`, in file order and never nil.
- `ConfigSpec` classifies `renameToolsKey` `configengine.CategoryGuard`, so a local override holding it is refused, exactly as for `verify_tiers`.
- `Load` resolves it through `stringList` and refuses an entry holding a line break with `unusable(renameToolsKey, …)`, the same check `verify_tiers` gets, so every content failure names `rename_tools` and ends with the `lyx config language` way forward.
  An empty `dirs.Board` returns `RenameTools: []string{}` beside the other two empty lists; an absent key loads the template's empty list.
- `template.yaml` gains `rename_tools: []` with a comment saying each entry names the hub's structure-aware rename tool for one language and how to run it, rendered verbatim into the Webster implementer prompt, the empty list meaning no tool, so a plan with a symbol `Rename` pair is refused.
  `ConfigTemplate`'s doc names the third empty list.
- `VerifyTiersValue` keeps its name and body; its doc says it renders the value every `verify_tiers` and `rename_tools` marker takes, so one renderer serves both lists.
- `doc.go` describes the three keys, and carries a Go hub's example value built on `gopls rename -w` (for each symbol pair, `gopls rename -w <file>:#<offset> <new name>` from the worktree root), and states that a Python-based tool needs shuttle's `deny_python` off, since the shell guard's python row refuses it otherwise.
- `refusal-spec.md`: the `config file content invalid` row in `## webster` and the `language.yaml invalid` row in `## shed` name `rename_tools` beside `verify_tiers` wherever they list the keys and the line-break check.

Tests:
`TestLoad_ResolvesSkills` gains a `wantRenameTools` column asserted non-nil and equal on every row, and a row listing all three keys that keeps `rename_tools` in file order;
`TestLoad_InvalidFailsNamingKey` gains rows `rename_tools scalar value`, `rename_tools empty entry`, `rename_tools entry with a line break` and a local override holding `rename_tools`, each naming `rename_tools` and marked unusable, and its no-key branch also checks the error names no `rename_tools`.

**ImpactSummary:** one new key on a loader every `language.yaml` consumer already calls; the existing two keys and their callers are unchanged.
