```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1–F3, F5–F11, and F4's carried-out parts).

No BLOCKING or MEDIUM findings. I built cases against F2, F4, F7, F8, F9 and F10, and two held up as LOW.

1. **LOW — F7, `packUnits` can leave a piece ending in a bare file header.**
   - Changed text: `boundedUnits` emits the header as its own unit (`diffUnit{text: header}`), and `packUnits` appends units while they fit.
   - Case: a package holds a small file A, then a large file B whose hunks do not fit.
     - B's header unit fits after A's text, so it is appended to the current piece.
     - B's first hunk then misses the cap, so the piece is flushed.
     - That piece ends with B's header and no hunk. The next piece repeats the header and starts B's hunks.
   - The text is not lost or wrong. The header sits orphaned at the end of a piece, which is mildly misleading.
   - Evidence: the `packUnits` loop flushes only when the next unit misses the cap. `boundedUnits` marks no tie between the header unit and the unit after it.
   - No test row pins this. `bigFile` is the only file in its package, so the case never arises.

2. **LOW — F7, the doc claim is wider than the code.**
   - Changed text: the `writeDiffPieces` comment, and the new `internal/burlerengine/doc.go` sentence "Each piece of a split file after its first opens with the file's header".
   - `continuationHeader` returns "" when the file has no `@@ ` line, as with a binary patch. It also returns "" when the pre-hunk text passes half of the cap.
   - Those continuation pieces open with no header, so the stated guarantee does not hold for them.
   - The `LineOverTheCap` rows rely on this fallback: they pass `wantJoined` only because the header exceeds half the cap.
   - The doc should say "when the header fits half a piece".

Proposals that held, with the cases I tried:
- **F2:** with 40,000 bytes and 1,500 lines, a dense base85 piece stays near 20,000 tokens, and the line-number prefixes do not push it past 25,000.
- **F8:** a quoted path, a tab-ended path and `/dev/null` fall through correctly. A binary section falls back to the header parse, which the review's own fix did not cover either.
- **F9:** the left-cut names stay unique because of the sequence prefix.
- **F10:** the rune-boundary cut is correct, and the `cut == 0` fallback only triggers on invalid UTF-8.
- **F4:** `CodeDiff` is wired at the single site where `CodeSubject` was, so the set of rows that stage is unchanged.

Tool calls: none; my context already held every changed file.
