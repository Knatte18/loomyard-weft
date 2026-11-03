```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1 code and test change (the hyphen skip in `rewriteCardMentions`, the fixture framing line), run step by step.

No findings.

- **Skip branch:** take `Run \`lym loom insert-card 3 slug\` to add one.` `cardMentionRe` matches `card 3` because `\b` is true between `-` and `c`. `span[0] > 0` holds and `masked[span[0]-1]` is `'-'`, so the span is skipped. It sets neither `kept` nor `pairs`, and `last` is untouched, so the line is emitted unchanged and unlisted.
- **Index safety:** `masked` is `line` with card ids blanked to the same length, so the index is in range. `span[0] > 0` guards the zero case.
- **Neighbouring input:** a reference preceded by an en or em dash (`–card 3`, `—card 3`) is not skipped. Its preceding byte is the last byte of a multi-byte UTF-8 sequence (`0x93` or `0x94`), not `'-'`. A space-separated range such as `cards 1–3` is unaffected.
- **Test against the defect:**
  - **Rows that fail with the defect in place:** the line holds `insert-card first+2`, which is 3. With the skip removed, the rows "insert at a middle position" and "insert at the third position" move card 3 to 4. The rows "remove at a middle position" and the plan-starting-above-1 rows move it as well. `framedAfter` would then hold `insert-card 4`. It would differ from `prose.Replace(framedBefore)`, since no replacer key matches this line. That fails the overview equality assertion.
  - **Rows that pass either way:** the "insert one past the last card" rows and "remove the last card" leave card 3 in place, so they pass with or without the skip.
  - **Coverage of the premise:** the moving rows cover it.
- **Expected-value maps:** the replacer keys are whole lines, and the new paragraph matches none of them. `removeSecondMentions` and the other mention lists are unaffected because the skipped line is never listed. The gate passed all tier0 targets.
- **Other callers:** `rewriteCardMentions` has one caller, `renumberReferences`, and its signature is unchanged.

Tool calls beyond the context: none.
