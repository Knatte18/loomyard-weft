```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the correctness lens over the F1–F8 change set, tracing a concrete input through each changed branch and test.

No findings.

What I traced:
- **F1, `Set` and `Remove`:**
  - A held shipped slug skips `refuseShipped` and edits or removes normally.
  - A new non-shipped slug passes `refuseShipped` and reaches the all-fields check.
  - A new shipped slug is refused, and so is `Remove` of a shipped slug the store lacks.
  - The `SetRefusals/ShippedSlug` row still matches because the message still contains `--layer lym`.
  - The new collision tests run those same paths. The CLI test's second `remove` hits `index < 0` and gets "ships with lym". The old `RemoveRefusesAShippedSlug` test would have failed, and it was correctly rewritten.
- **F2, `ReadLayers`:**
  - A load error now sets `OverrideWarning` and an empty `Overrides`, and returns `nil`.
  - The error text names `heddle-lym-override.json`, which the tests assert.
  - Without the fix the malformed row would have failed `mustRender`.
  - The blank-text row would have rendered an empty section, so it fails without the `TrimSpace` guard.
  - The `list` warnings field is covered by the new CLI test.
- **F3:**
  - The non-parallel test swaps `shippedStore` with a `t.Cleanup` restore, so paused parallel tests never see the swap.
  - Expected strings check out for all three rows:
    - Override on gives "tuned prompt" and "tuned file text".
    - Override off and no config both give the shipped texts.
    - Prompt order is shipped before repository, although the repository entry's order is lower.
  - Breaking the shipped-first order in `passagesOf` or dropping the override in `Show` or `Passages` makes it fail.
- **F5:**
  - Each `decodeShipped` panic row matches its panic message.
  - The no-config row fails if the default were on.
  - The stale-slug row passes by construction and only guards against a regression.
- **F4, F6, F7, F8:** the `Claims` assertion, the pinned batching entry and the doc and help-text edits are consistent with the code.

Tool calls: none; my context already held every changed file.
