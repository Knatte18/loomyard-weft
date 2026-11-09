```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the two changed test files. I found no defects.

**Per function or branch, with the case I ran**

- `loadTemplateConfig`: I ran it with the shipped template seeded alone. It loads cleanly. A load error ends in `t.Fatalf` with the error text, so nothing is swallowed. Correct.
- `TestLoadConfig_Loads`, `template defaults` subtest:
  - I ran the sorted map keys through `FieldByName` and `DeepEqual`.
  - Each map key exists as a `Config` field, and the 41 entries match the old `templateConfig()` literal.
  - `DiscussionMaxBounces` and the other `*int` fields compare by dereference, so `intPtr(2)` equals the loaded pointer.
  - A renamed field gives `IsValid()==false`, so the test errors loudly instead of skipping, and `continue` skips only that entry.
  - Zero defaults such as `DiscussionInteractive: false` are still pinned, because they are present in the map.
  - Correct.
- Table cases, `want := templateLoad; tt.mutate(&want)`:
  - The struct copy is shallow, but every `mutate` assigns new slice or pointer values and none edits one in place. The shared base is therefore not corrupted, and parallel subtests only read it.
  - A "retired key ignored" case with an empty `mutate` is still a real check, because the whole struct must equal the template load.
  - Some mutates set a field to the template's own value, for example `ReviewMaxBounces=3` and `DiscussionFixerFan={"fix-discussion"}`. They were already like that under the old literal. The sibling field in the same case (`ReviewHardMaxBounces` 3 vs 4, the other fan keys) carries a real change, so no case is vacuous overall.
  - `wantSoft` is unchanged, and the soft-cap numbers still derive from the loaded values.
  - Correct.
- Hub-stack subtests: the same `templateLoad` base applies. The `reviewer_model` override and the `review_max_bounces: 2` case both differ from the template, so they assert something. Correct.
- `TestModules_EveryTemplateKeyClassified`, `specFunc`:
  - `m.Spec` is a plain package function in every `Module` literal I saw (`pkg.ConfigSpec`).
  - `FuncForPC(...).Name()` therefore returns a stable name such as `…/loomengine.ConfigSpec`. A closure or method value would give a `.func1` or `-fm` name, but none is registered. The `Lookup` test fixtures use closures, but they do not go through this test.
  - A nil `Spec` would already panic at `m.Spec()` on the line above, so the new line adds no new failure mode.
  - Correct.
- `keyListWayForward` and `keysOutside`:
  - With `got=[a,b]` and `want=[a]`, `added = keysOutside(want, got)` gives `[b]`, and the message says "add b to the list".
  - With `got=[a]` and `want=[a,b]`, `dropped = keysOutside(got, want)` gives `[b]`, and the message says "drop b".
  - So the directions match: the expected list gains what `got` holds and `want` lacks.
  - An order-only mismatch gives `steps` empty, so the message is "sort … in this test".
  - A nil `want` (module absent from the map) gives `added = got`, which reads correctly.
  - A length difference caused only by a duplicate in `got` would also produce the "sort" message. That is harmless and not realistic.
  - Correct.
- Unclassified-unit and orphan-entry messages: these name `specFunc`, the unit or entry, and the module, and the edit target is the Categories map the `Spec` function returns, which is right. Correct.

**Findings:** none.

**Tool calls (1):** a `sed`/`grep` on `internal/configreg/configreg.go` and `internal/loomengine/config.go`. It checked that every `Spec:` value is a plain package function and that the `Config` fields exist for the `templateDefaults` names. My context held only part of the registry and no `Config` struct. The `sed` half failed on my own `cut` syntax, so only the `Config` field list printed. The registry coverage rests on the first 20-odd registry lines I had already read, where every `Spec:` value is a plain package function.
