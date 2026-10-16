# Card 1 — language-verify-tiers

**Edit:**
- `internal/languageconfig#Load` -> `func Load(dirs configengine.Dirs) (Language, error)`
- `internal/languageconfig#config`
- `internal/languageconfig#ConfigSpec`
- `internal/languageconfig#unusable` -> `func unusable(key, detail string) error`
- `internal/languageconfig/template.yaml`
- `internal/languageconfig/doc.go`
- `internal/languageconfig_test#TestLoad_ResolvesSkills`
- `internal/languageconfig_test#TestLoad_InvalidFailsNamingKey`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_LanguageSkills`
- `internal/webstercli#websterCLI.wireHub`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/configreg#Modules`
- `internal/languageconfig#ConfigTemplate`
- `cmd/lyx/testdata/test-budget.yaml`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Create:**
- `internal/languageconfig#Language`
- `internal/languageconfig#verifyTiersKey`
- `internal/languageconfig#stringList`
- `internal/languageconfig#VerifyTiersValue`
- `internal/languageconfig_test#TestVerifyTiersValue`

**Intent:** `language.yaml` gains its second key, `verify_tiers` (D5), and `Load` resolves both keys into one value.

- `Language` holds `Skills []string` and `VerifyTiers []string`, each in file order; `Load` returns it, and the empty-board degrade returns both lists empty and non-nil.
- `config` gains `VerifyTiers yaml.Node` under the `verify_tiers` tag.
- `stringList` holds the shape checks `skills` runs today (a sequence of `!!str` scalars, no empty entry, no surrounding whitespace), each failure through `unusable` naming the key it was given.
  `Load` runs it over both keys unconditionally.
  An existing hub file without `verify_tiers` reaches `Load` holding the template's empty `verify_tiers` list, because configengine's load fills missing template keys through `yamlengine.FillMissing`, so it loads unchanged.
  `Load` additionally refuses a `verify_tiers` entry holding `\n` or `\r`, through `unusable(verifyTiersKey, …)`, so every entry renders as one bullet line.
- `unusable` takes the key it names; a configengine content failure or an unparseable file has no single key, so `Load` passes an empty key and `unusable` then writes `language config: <detail>` with no key segment.
  The marking (`configengine.ErrInvalid` wrapping `ErrUnusable`) and the `wayForward` clause are unchanged.
- `ConfigSpec` classifies `verify_tiers` `configengine.CategoryGuard` beside `skills`, so a local override holding it is refused by configengine, whose message names the key.
- `template.yaml` gains `verify_tiers: []` with a one-line comment: each entry is one rule of the hub's verify tiers (their order, which tiers are compiled rather than run and why, a nested-module form, the raw-run limits) that the plan writer, Plan-Review and implementer prompts render verbatim, and the empty list is the neutral default.
  An existing hub's file gains the key only through `hubreconcile.Ensure` (`PATTERN-binary-change-reconcile`); no card touches a hub file.
  `ConfigTemplate`'s doc comment, which calls the template "an empty skills list", names both empty lists.
- `VerifyTiersValue` is the one marker value every site renders (D6): `none` for an empty list, else each entry as a `- <entry>` line, joined by `\n`, entries verbatim and in order, never parsed.
  It lives here so the plan, rubric and implementer fills share one rendering.
- `doc.go` describes the second key, its Guard class, its refusals (line break included), the empty default, `VerifyTiersValue`, and, as D7 prescribes, this hub's rules as the example value of a Go hub (the four bullets of D7), set by the operator with `lyx config language` from the prime in the editor.
  "lyx hard-codes no language" stays true and now covers both keys.
- `contracts/specs/refusal-spec.md`'s `config file content invalid` and `language.yaml invalid` rows widen from `skills` to both keys: a `verify_tiers` value that is not a list of strings, an entry that is empty, carries surrounding whitespace or holds a line break, and a local override holding either key; the way forward is unchanged (`PATTERN-refusal-way-forward`).
- `docs/overview.md`'s module-table line for `internal/languageconfig/` names the verify tiers beside the skills.
- Callers: `loomCLI.wire`, `websterCLI.wireHub` and `burlerCLI.wireHub` read `.Skills` where they used the returned slice; `loomCLI.wire` keeps the whole `Language` in a local for the later cards.
  The comment above `loomCLI.wire`'s `languageconfig.Load` call (`internal/loomcli/wiring.go`) says the error already names the failing key and its own way forward.
  `configreg`'s `Modules` row passes `languageconfig.Load` to `checkLoad`, which is generic over the result: the row's text stays as is and is listed only because it references the re-signed `Load`.

Tests:

- `TestLoad_ResolvesSkills` gains a `wantTiers` column: every existing row wants the empty tier list (the template row proves the template's `verify_tiers: []`); a new row lists two tiers and some skills and wants both lists in file order; a row with a skills list and no `verify_tiers` key wants the empty tier list, which pins the template fill.
  Every row also asserts both returned lists are non-nil, which `slices.Equal` alone does not, so the no-board row pins the non-nil degrade.
- `TestLoad_InvalidFailsNamingKey` gains a `key` column, the key the error must name beside `lyx config language`.
  The unparseable-yaml row has an empty `key` and asserts the error names neither `skills` nor `verify_tiers`, pinning `unusable`'s no-key form.
  New rows refuse `verify_tiers`: a scalar value, a non-string entry (`[1]`), an empty entry, leading whitespace, trailing whitespace, an entry holding `\n` (`["a\nb"]`), an entry holding `\r` (`["a\rb"]`), and a local override holding `verify_tiers`; each is a content failure naming `verify_tiers`.
- `TestVerifyTiersValue` is table-driven: nil and empty give `none`; two entries give two bullet lines in order; an entry holding `{{.x}}` is rendered verbatim.
  The package's row in `cmd/lyx/testdata/test-budget.yaml` rises by one for it.
- `TestWire_LanguageSkills` gains a `wantKey` column that replaces its fixed `"skills"` check, `skills` on the existing error row.
  Its listed row's file also holds a two-entry `verify_tiers` list, and the row struct gains a `wantTiers` column holding those entries, asserted against the `Language` that `loomCLI.wire` keeps.
  It gains a row whose file holds a valid `skills` list and `verify_tiers: [""]`: `wire` halts with an error naming `verify_tiers` and `lyx config language`, the real spawn path the acceptance criterion names.

**ImpactSummary:** `Load` re-signed with three production callers (loom, webster and burler CLI wiring) switched to `.Skills`; `configreg` compiles unchanged; the only behavior change is the new key and its refusals, recorded in the refusal spec and the module table.
