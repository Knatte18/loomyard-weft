```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- **F1, `anyLevelHeadingPattern`:**
  - `// ## <120 chars>` gives counted text `## ...`, which `^#{1,6}(\s|$)` matches, so the line is exempt.
  - The old `^#(\s|$)` does not match `##`, so the new `a level-two heading is exempt` row would fail with the defect in place.
  - `headingPattern` is untouched, so the wrapped-break rule still uses it in `lint.go`.
- **F2, per-occurrence `known` counts:**
  - A base holding one `atLimit` and a new text holding two copies gives `known=1`. The first copy decrements it, and the second reaches the length check and is flagged at line 4.
  - With the old flat set the second copy was skipped, so the new row fails with the defect in place.
  - The `a long line moved within the file` row is a regression guard. It passes under both versions, which is acceptable.
  - Decrementing on short lines is harmless, because the literal is the key and a short line's literal is never a long line's.
- **F3, `TestFindFileFindingsOrdersByLine`:**
  - Case one is `source(atLimit, "// a wrapped", "// line.")`. `atLimit` is `"// "` plus `proseOfLength(120)`, which is exactly 120 characters of text, ending in `.`.
  - The break from line 3 to line 4 is a sentence end, so it passes. The break at line 4 is flagged as a wrapped break, and line 3 is flagged as a long line.
  - `findNewWrappedBreaks` appends its finding first, so the order is [4, 3] before the sort and [3, 4] after it. Removing the stable sort fails this row.
  - Case two (wrapped break at line 3, long line at line 5) is already in line order before the sort. It passes either way and only pins that both rules appear.
  - The expected `Finding` values (`File "p.go"`, `Text` as the trimmed literal) match what the code builds, and `Finding` is comparable, so `slices.Equal` compiles.
- **Callers:** `findNewLongLines` keeps its signature, `flaggedLines` and `findFileFindings` are unaffected, and the parent's gate run (`commentlint`, `gateslot`, `websterengine`) was green.

Further tool calls: none, because the context already held every changed file and the gate result.
