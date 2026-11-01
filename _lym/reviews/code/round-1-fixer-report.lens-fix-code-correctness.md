```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the code in the fix (`store.go` Decode/Load, `verbs.go` setCmd, `store_test.go`) for correctness.

**Walked inputs**
- `Decode([]byte("{\"entries\": ["))`: `decodeBytes` returns the JSON error, and `Decode` wraps it as `heddle.json holds malformed JSON: …; way forward: fix the file in a new commit on the local branch`. The `Replayer.Apply` callers prefix `merge base`, `parent` or `commit`, so the failing side is named. Correct.
- `Load(dir)` with the same bytes: the message is `heddle.json does not decode: …`, with the `git -C <dir> checkout HEAD -- heddle.json` advice. Correct, and the existing Load test still matches it.
- `Decode` on empty or whitespace bytes returns an empty store through `decodeBytes`, and `Load` on an absent file is unchanged.
- `heddle set a --targets ""` on an existing entry: `Changed("targets")` is true and `splitTargets("")` returns nil, so the fix sets `[]string{}`. `Set` sees a non-nil empty slice and `candidate.Targets` becomes empty. `ValidateEntry` then refuses it with `no target … give at least one of … in --targets`, and the store is unchanged. Correct. A new entry given `--targets ""` takes the same route.
- The only caller of `splitTargets` is `setCmd`. The `decodeFrom` removal and the `Decode` change compile, and the gate passed for both packages.

**Findings**

1. LOW, `internal/heddlecli/verbs.go` (the `patch.Targets = []string{}` branch in `setCmd`). Nothing tests the CLI half of F1.
   - The new test row `ExistingWithEmptyTargets` calls `Store.Set` with `Patch{Targets: []string{}}`. That already reached `ValidateEntry` before the fix, so the row only fails with the old `--target` wording.
   - With the `verbs.go` branch reverted, `--targets ""` yields a nil patch again and silently changes nothing, and every test still passes.
   - `internal/heddlecli/cli_integration_test.go:221` already runs `text set … --targets no-such-target` through `requireRefusal`. A row there with `--targets ""` on an existing slug, expecting `--targets`, would fail with the defect in place.

2. No other findings.
   - The `NewWithoutTargets` row now discriminates, since `--target` is not a substring of `--targets`. Before, it passed either way.
   - The `Decode` test fails with the old code, which carried `<Bolt checkout>` and `checkout HEAD` and lacked `new commit`.

**Tool calls:** none. Everything came from the context.
