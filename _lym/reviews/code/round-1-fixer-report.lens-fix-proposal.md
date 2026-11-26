```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer proposals (F1–F7) that the fix carried out.

Proposals challenged: F1, F2, F3, F4, F5, F6, F7.

1. **MEDIUM**: `contracts/specs/loom-discussion-spec.md`, the moved and rewritten sentence "A bullet runs on through the lines after it up to a blank line or the next bullet, and an indented line always continues it." (F6, with F1's wording).
   - **Summary:** "an indented line always continues it" is false, and the sentence leaves a fence out of the things that end a bullet.
   - **Evidence:** `fencedLines` follows the top-level fence rule the same spec states, where an opening run may have up to three leading spaces. So in `- a\n  ```\n  x\n  ```` the indented lines are fenced. `bulletLines` then sets `inItem = false` for `fenced[i]`, and `SectionBullets` calls `flush()` on `fenced[i] || skipped[i]`. An indented line inside such a fence, or inside a carry-over block (`SectionBullets` only), does not continue the bullet.
   - **Origin:** the wording before this fix ("up to a blank line, the next bullet or a heading") made no "always" claim.

No other findings:
- **F1:** the ATX and HTML start-condition patterns match CommonMark's conditions for lines that interrupt a paragraph. A placeholder such as `<title>` ends a bullet, as it does in markdown.
- **F2, F5, F7:** no breaking case found.
- **F3:** "records" is one word, `validate-description` passed, and the refiling claim holds.

Outside the rubric, noted only: the spaced thematic breaks `- - -` and `* * *` match `isBulletLine` first, so they never reach `thematicBreakLine` and still read as a bullet with the text `- -` or `* *`. This behaviour predates the fix, and F4's row (`---`) is handled.

Tool calls: none.
