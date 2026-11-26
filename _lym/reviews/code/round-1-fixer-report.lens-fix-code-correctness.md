```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed.

1. **MEDIUM**: `contracts/specs/loom-discussion-spec.md`, the new sentence "an indented line always continues it".
   - The code does not hold this. A fence opens on a line with up to three leading spaces, and both `SectionBullets` (`case fenced[i] || skipped[i]: flush()`) and `bulletLines` (`if fenced[i] { inItem = false }`) end the bullet there.
   - Concrete input: `- a\n  ```\n  D2\n  ```\n  more` reads as the bullet `a` alone. The indented fence lines and `more` do not continue it, and `D2` stays uncovered.
   - Indented lines inside a carry-over block end it too.
   - The same sentence's "up to a blank line or the next bullet" also leaves out the fence.

2. **LOW**: `decisionids.go`, `thematicBreakLine` and the "unless it is … a thematic break" wording in the `continuesItem` doc, the `SectionBullets` doc and the spec.
   - The spaced forms `- - -` and `* * *` never reach `continuesItem`, because `isBulletLine` matches them first in both readers. The `-` and `*` space-between branches of the regexp are therefore dead.
   - `## Open risks\n\n- a\n* * *\n` reads as `["a", "* *"]`, so raddle coverage would require a part to carry `* *`.
   - The behaviour for these forms predates the fix. F4's proposed "with only spaces between" is only effective for `_ _ _`.

The rest holds on the inputs I traced by reading; I ran nothing:
- `#123 b`, `####### x`, `<slug> b`, `<https://x>`, `<path>` and `</pre>` continue the bullet.
- `# h`, `---`, `___`, `> q`, `<!-- n -->`, `<details>` and `<title>` end it.
- By the same reading, each new `TestSectionBullets` row fails with its defect restored.
- The changed `TestSplicePart` criterion fails at `requireBullets` under the indented-only rule.
- A spliced record reads back the same text, because `renderBullets` indents continuation lines.

No further tool calls made.
