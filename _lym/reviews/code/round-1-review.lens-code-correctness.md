```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the diff (shipped.json entry, `store_test.go` change, stencil edits).

**Per function/branch (case run, verdict):**
- `decodeShipped` (`shipped.go:36`) loading the new entry: slug `lym-plan-edits-before-gate` matches `[a-z0-9-]+`. The text is non-blank, the target `claude-file` is a known target, and the slug differs from `lym-batching`, so there is no panic at package load. Correct.
- `Store.Ordered` (`store.go:219`) with orders 10 and 20: the new entry sorts after `lym-batching`. Correct.
- `Layers.Ordered` (`shipped.go:72`) with empty Overrides: both shipped entries are emitted, shipped first. Correct.
- `ShippedEntriesArePinned` (`store_test.go:169-193`), happy path: the lengths match (2 and 2), and both slugs and targets match. The batching text is compared exactly, and the new entry's text is deliberately unpinned. Correct.
- Same test, drift cases: an added entry makes the `Fatalf` length check fail first, so the loop never indexes out of range. Correct. An empty `Targets` slice trips the `len(got.Targets) != 1` check before `Targets[0]` is read, because `||` short-circuits. Correct.
- Other tests that use `shippedStore.Ordered()[0]` (`render_test.go:296,305,345,371`, `store_test.go:149,208,220`, `override_test.go:14`): index 0 is still `lym-batching`, so they are unaffected. Correct.
- `render_test.go:80`, which loops over all shipped entries, and `heddlecli/cli_integration_test.go:159`, which loops over `(Layers{}).Ordered()`: both iterate generically and are not broken by an extra entry. That is from the diff and the grep output only. I did not open their bodies, so any hard-coded count or expected output there is unchecked.
- Stencil edits (text only, no code path): the removed lines leave no dangling reference. The `webster-body-implementer.md:58` "in one message" lines and `edit-directive.md` remain. They are about retired-name sweeps and one file's edits, so they are card-specific, not duplicates of the shipped rule. The fork-spawn "one message" lines in `twill-directive.md:13` and `burler-fix-self-check.md:10` are not batching of reads. No correctness defect.

**Findings:** none.

**Tool calls (3):**
1. Read `internal/heddleengine/shipped.go`: the manifest lacked the shipped-layer decode and validation path.
2. Grep and read `internal/heddleengine/store.go` (`Ordered`): to confirm the ordering semantics.
3. Grep `internal/heddleengine/shipped.json` orders and read `ValidateEntry`: to confirm `lym-batching` has order 10 and the new entry passes validation.
