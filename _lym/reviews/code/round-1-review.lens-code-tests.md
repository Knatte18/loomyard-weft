```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** For each behaviour the diff adds or changes, the test that pins it, or none.

| Behaviour | Pinning test |
|---|---|
| Dotted key sets one leaf, keeps siblings, order and head comments | `TestSetEntries` row "a dotted key sets one value inside an entry and keeps the rest" (`set_test.go:885`) |
| Missing mappings created; `0` written as a number; absent document | Row "creates the mappings it lacks and writes a number as a number" (`set_test.go:891`) |
| String containing brackets stays a string; empty value is `""` | Row "no number stays a string, and an empty value is an empty string" (`set_test.go:896`) |
| `{` value on a dotted key is a mapping at the key's own depth | Row "starting with a brace is a mapping at the key's own depth" (`set_test.go:901`) |
| Dotted key through a scalar is refused, naming the path and key | Row "through a scalar is refused" (`set_test.go:906`) |
| Empty segment is refused, including the empty key `""` | Rows "an empty name is refused" and "dotted key with an empty segment is refused" (`set_test.go:879`, `912`) |
| Undotted refusal names the dotted form | Row "a value that is not a JSON object is refused" (`set_test.go:873`) |
| Undotted path unchanged | Existing rows (`set_test.go:851-872`) |
| `[` value on a dotted key is written as a flow list | None |
| Malformed flow value on a dotted key (`{a: 1`) is refused | None |
| `true`, `null` and `~` take their own tags | None. Only `0` is exercised, so a regression in `scalarSetTag("!!null", …)` for non-numeric scalars would pass. |
| A dotted key through a null value turns it into a mapping | None. The `mappingAt` null branch has no `SetEntries` row. |
| A set leaf keeps its comments | None. The dotted rows set no leaf that carries a comment. |
| Dotted key reaches `SetEntries` through `configengine.Set` or the `profiles` CLI | None. The `jsonEntries` row (`configengine/set_test.go:350`) and the `TestDispatchSetProfiles` rows (`configcli_test.go:1191-1212`) are all whole-entry sets. |

**Findings**

- **T1, LOW, scope** — `internal/yamlengine/set_test.go:906`. No test pins a dotted `[` value (flow list) or a malformed `{`/`[` value on a dotted key. `setEntryValue` has separate branches for both. Fix: add a row `docs.darn.fan` = `[a, b]` that wants `fan: [a, b]`, and a row `docs.darn` = `{a: 1` that wants an error naming the key and containing "double-quote".
- **T2, LOW, scope** — `internal/yamlengine/set_test.go:896`. The scalar tag resolution for a leaf that is not a number is unpinned: `true`, `null`, `~`, and a number-like string such as `007`. The docs promise "a scalar under the type its text spells". Fix: add rows such as `docs.darn.on` = `true` that wants `on: true` and `docs.darn.x` = `null` that wants `x: null`. Without these, a regression to always-string would pass.
- **T3, LOW, scope** — `internal/yamlengine/set_test.go:906`. No row for a dotted key through a null value (`docs:\n  darn:\n`) becoming a mapping, nor for a replaced leaf keeping its line or head comment. The `SetEntries` doc comment promises "a set key keeps its comments". Fix: add two rows, one for an existing `darn:` null and one for a commented leaf being overwritten.
- **T4, LOW, scope** — `internal/configengine/set_test.go:350` and `internal/configcli/configcli_test.go:1191`. The dotted path through `configengine.Set` is untested: the `setEntries` doc comment and `validateOverride` on a dotted merge, and the `profiles` CLI that issue #94 targets. Fix: add one `TestDispatchSetProfiles` row, `--set light.loom.code_max_bounces=0`, that wants the profile's other keys kept and the value changed. A second row, with `--layer local` or `--layer machine`, would pin the refusal; check the existing local-layer row first to avoid duplicating it.

No test in the diff duplicates existing coverage: the six new rows each pin a distinct behaviour.

**Tool calls (2 of 6 used)**
1. `git diff --stat` on test files, plus `grep` for `jsonEntries` in `configengine/set_test.go` and `TestDispatchSetProfiles` in `configcli_test.go`: to confirm the diff touches only `yamlengine/set_test.go` and that the other layers' tests have no dotted rows. The manifest lacked those tests.

I made one Bash call and no other tool calls.
