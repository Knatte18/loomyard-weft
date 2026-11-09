```yaml
lens: code-correctness
outcome: reported
```

Task: review the correctness of the code the diff changes (`SetEntries`, `setEntryValue`), one concrete input per branch.

**Per-branch list**

1. **Empty-segment guard** (`set.go:859`): inputs `""`, `"a."`, `".a"`, `"a..b"`. All fail before any write with the message naming the key. The `""` case still contains "entry name", so the existing row passes. Correct.
2. **Undotted whole-entry path** (`set.go:868-879`): input `light=[1]` is refused naming the dotted form. Input `light={...}` is unchanged. Correct. A dotted profile name no longer works: `v1.5='{...}'` used to create a top-level entry `v1.5` and now sets `v1: {5: ...}`. `profile name` validation only rejects an empty name (`runprofile.go:158`), so a dotted name was legal before. See F1.
3. **`setEntryValue`, scalar default branch**:
   - `0`, `true` and `1e3` take `!!int`, `!!bool` and `!!float` and marshal plain. Correct.
   - `null` and `~` take `!!null`. Correct.
   - `sonnet[medium]` stays a string. Correct.
   - `a: b` and `- x` parse as non-scalars, so they get `!!str` and are quoted on output. Correct.
   - `# c` parses to an empty document, so `!!str`. Correct.
   - `"0"` with literal quote characters is stored as the literal text `"0"` (quotes included), because `Value` is the raw text and quotes are never stripped. A numeric-looking or bool-looking string cannot be written through a dotted key. This matches the existing `SetValues` leaf behavior and is not a regression. See F2.
   - `" 5 "` gets tag `!!int` with value `" 5 "`. I did not run the marshal, so I don't know whether it emits valid YAML. `setShapedValue` has the same pattern, so this is not new.
4. **Empty value** (`set.go:900`): `""` becomes a `!!str` that marshals as `""`. Correct. A whitespace-only value is not caught by the empty case and falls to the default branch, where `scalarSetTag` returns `!!str` and the value is quoted. Correct.
5. **Flow branch**:
   - `{"loom":{"code_max_bounces":0}}` is written as a block mapping at the key's own depth, with nested flow kept and scalar quoting cleared. Correct, and the test row confirms it.
   - `{}` marshals as `{}`. Correct.
   - `[a, b]` stays a flow list. Correct.
   - An unterminated `{a: 1` or trailing text `{a: 1} x` returns the `openMapEntryNode` refusal naming the key. Correct.
6. **Path walk via `parentMappingOf`/`mappingAt`**:
   - A missing mapping on the way is created.
   - A null on the way becomes a mapping.
   - A scalar on the way gives "docs.darn holds a scalar, not a map", wrapped with the full key. Correct.
   - A top-level scalar, `light: 5` with `light.x`, is refused the same way. Correct.
   - On any error `SetEntries` returns `nil` and the parsed doc is discarded. `configengine.setEntries` writes only after success, so a mid-path mutation never reaches disk, even with several pairs. Correct.
   - A non-empty flow parent stays flow when a mapping is added, which is valid YAML.
7. **`setEntry` replace** (`layers.go:399`): comments of the old value are kept. Setting `docs.darn=5` over an existing mapping silently replaces the mapping. This is intentional and matches the shaped-set behavior.
8. **`configengine.setEntries`**: an absent base file starts from the template and then applies the dotted set. Validation of local and machine layers runs on the merged bytes. Correct.
9. **Callers**: the only `JSONEntries` spec is `runprofile`, so only the profiles module is affected, with no other existing write changed. `configcli` splits each `--set` pair on the first `=` (`configcli.go:194`), so `=` inside a value is fine.

**Findings**

- **F1 — LOW — `design` — `internal/yamlengine/set.go:859-867`**
  - A dotted top-level name can no longer be written. `--set v1.5='{...}'` used to add the entry `v1.5` and is now read as a path.
  - The runtime accepts such a name, since `runprofile.go:158` only rejects an empty one.
  - If dotted profile names are meant to be unsupported, the docs or refusal should say so. Otherwise no whole-entry route remains.
  - This is a behavior change the board entry does not mention, and `v1.5` was a legitimate pre-existing top-level name.
- **F2 — NIT — `design` — `internal/yamlengine/set.go:903-904`**
  - A dotted scalar is always typed by its own text, so there is no way to write the string `"0"`, `true` or `null`. Literal quotes in the value are stored as part of the string.
  - This is the same limit as `SetValues` leaf sets, and the doc comment says "the tag its own text resolves to". It is only worth noting if string-typed profile keys exist; I did not check.

I found no wrong result and no swallowed error on any branch I ran.

**Tool calls (2)**
1. Grep for profile-name validation in `runprofile`, `boardcli` and `boardengine`, to check whether a dotted whole-entry name was legal. The context lacked it.
2. Grep of `configcli` for the `key=value` split (found at `configcli.go:194`), to confirm how a value containing `=` reaches `SetEntries`. The context lacked it.
