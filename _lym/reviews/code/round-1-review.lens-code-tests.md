```yaml
lens: code-tests
outcome: reported
```

The code-tests lens on the diff `3a854635..HEAD`: every behaviour the diff adds or changes has a test row that would fail if it were broken.

**List: per behaviour, the test that pins it**

1. **The template defines the `code-scope` lens.**
   - Pinned by `TestConfigTemplate_DecodesAndIsSelfConsistent`: `wantLenses = 38` (internal/lenses/config_test.go:359).
   - The `wantOrders["review-code"]` row also pins it, through `ResolveFan` refusing an undefined lens (config_test.go:392, :399-402).
2. **The template's `review-code` fan holds four lenses in base order, with `code-scope` last.**
   - Pinned by the count map `"review-code": 4` (config_test.go:374).
   - Also pinned by the `wantOrders["review-code"]` order check against `ResolveFan` (config_test.go:392, :408-412).
3. **`code-scope` is a procedure lens, so it may end in its list instead of the emphasis line.**
   - Pinned by `procedureLenses`, which is filled from the `review-` fans (config_test.go:403-407).
   - It then feeds `emphasisLineGaps(cfg.Lenses, procedureLenses)` (config_test.go:482).
   - Leaving `code-scope` out of the fan would make the emphasis-line check fail on its text.
4. **The csharp layer's `review-code` fan starts with the base fan, now including `code-scope`, and ends with `numerics`.**
   - Pinned by the subtest `layer csharp`: the base-prefix check (config_test.go:549-553) and the last-is-`numerics` check (config_test.go:566-574).
   - Dropping `code-scope` from csharp.yaml, or moving it after `numerics`, fails one of the two.
5. **Removing the four base `review-code` lenses empties the fan, and the emptied fan is refused when resolved.**
   - Pinned by the `TestCheckLoad` subtest "a remove emptying a fan passes the check and the fan is refused at resolution" (config_test.go:714-738).
   - Without `code-scope` in its `remove:`, the fan would still be held and the `held` check would fail.
6. **The `code-scope` prose: its two finding sentences, decision-id sources, goal-point fallback and forced-edit exemption.**
   - This is static prose that a review fork reads, so it is exempt under this lens.
   - Two of its sentences are asserted at config_test.go:441-444.

**Findings**

None under this lens.

Outside this lens: the new `code-scope` phrase assertion at internal/lenses/config_test.go:441-444 asserts static prose, which the rubric's test-economy bullet covers. The handler owns that call.

**Tool calls**

1. `grep` in internal/lenses/config_test.go for the emphasis-line and per-layer checks: my context held the test only up to line 470.
2. `sed -n 470,580p` of internal/lenses/config_test.go: to read the per-layer superset and csharp `numerics` checks the context lacked.
