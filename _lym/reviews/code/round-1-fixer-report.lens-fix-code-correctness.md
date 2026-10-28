```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Trace of the new row: the base index holds the word pair `b c` and the shape [2,2] line pair; the new text gives `x y`, not in `wordPairs`; `endsSemanticBreak` is false; unchanged is 2 of 4, so `2*2 > 4` is false and line 3 is flagged. With `>=` the row yields nil, so it fails with the defect in place. Test-only change, no caller affected.

Tool calls made: none.
