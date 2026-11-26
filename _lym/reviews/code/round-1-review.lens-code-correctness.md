```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff's changed functions and branches. I traced each one by hand from the inherited context.

**Cases run**

- `continuesItem("")` → false. Correct: a blank line ends the item.
- `continuesItem` on a whitespace-only line → false. Correct: both callers right-trim first, so the line arrives as `""`.
- `continuesItem("  more")` and `continuesItem("\tmore")` → true. Correct.
- `continuesItem("only inside the area.")` → true. Correct: this is the lazy continuation the task asks for.
- `continuesItem("### Note")` → false; `continuesItem("&lt;!-- lym:carry-over code --&gt;")` → false; `continuesItem("&gt; q")` → false. Correct.
- `continuesItem("#123 still holds")` → false. **Wrong.** Without a space after the `#` this is not an ATX heading, so markdown continues the item lazily.
- `continuesItem("&lt;https://example.com&gt; is the source")` → false. **Wrong.** An autolink does not open an HTML block. The same goes for an angle-bracket placeholder such as `&lt;slug&gt; is kept`: an HTML block of type 7 cannot interrupt a paragraph.
- `continuesItem("---")`, `("***")`, `("___")` → true. **Wrong.** A thematic break interrupts a paragraph and ends the list in markdown, but here it joins the bullet's text.
- `SectionBullets` on `- a\nb\n- c` → `["a\nb", "c"]`. Correct.
- `SectionBullets` on `- a\n#123 b\n- c` → `["a", "c"]`. The `#123` line falls to `default: flush()` and is dropped, so the bullet is cut. This is the reported defect, still present for this opener.
- `SectionBullets` with an unindented line after a closing fence, or after a carry-over block: `current` is nil, so the line is not absorbed. Correct.
- `bulletLines`, same inputs: the fenced line sets `inItem` false and the next line is not added. That matches `SectionBullets`.
- `bulletLines` on `- D1 holds,\nand so does D2.` → both lines are kept, so D2 is covered. Correct.
- `bulletLines` on `- D1\n&lt;b&gt; names D2`: the second line ends the item, so D2 counts as uncovered. Wrong, by the same opener defect as above.
- `bulletLines` on a non-bullet line before any bullet: `inItem` stays false and the line is not added. Correct.
- `bulletLines` vs `SectionBullets` on `- a\n  - b\nc`: both include `c`, as markdown's lazy continuation of the nested item does. Correct.
- Round trip of `"D6: x\nonly inside the area."` through `renderBullets` then `SectionBullets`: the continuation is written indented and trimmed back, giving the same text. Correct.

**Findings**

1. **MEDIUM, design.** `internal/discussionparser/decisionids.go:203`
   - **Summary:** `continuesItem` ends a bullet at any unindented line whose first character is `#` or `&lt;`. That is wider than "opens a heading or HTML block", which is what the doc comment (`decisionids.go:196`), `SectionBullets`' doc (`partsplice.go:38`) and the spec line (`contracts/specs/loom-discussion-spec.md:63` in the diff) promise.
   - **Effect:** a continuation such as `#123 …`, `#tag …`, `&lt;https://…&gt; …` or `&lt;placeholder&gt; …` is still cut from the bullet. `SectionBullets` drops it through `default: flush()` (`partsplice.go:70-71`), and `bulletLines` stops collecting (`decisionids.go:184-185`). This is the reported criterion cut and false `decision-not-in-acceptance`, reproduced for these openers.
   - **Fix:** exclude `#` only when it starts an ATX heading (one to six `#` followed by a space or the line's end). Exclude `&lt;` only when it opens an HTML block that can interrupt a paragraph (types 1–6, such as `&lt;!--`, `&lt;?`, `&lt;!X`, `&lt;![CDATA[`, or a block-level tag name).

2. **LOW, design.** `internal/discussionparser/decisionids.go:203`
   - **Summary:** an unindented thematic break (`---`, `***`, `___`) right after a bullet joins its text. Markdown ends the list there.
   - **Effect:** `SectionBullets` on `- a\n---` returns `["a\n---"]` (`partsplice.go:68-69`). That contradicts the "as markdown reads a lazy continuation" claim at `decisionids.go:196`.
   - **Fix:** treat a thematic-break line as ending the item.

Outside this lens: absorbing unindented prose directly under a section's last bullet changes the stored bullet text, which existing raddle proposals match exactly. That belongs to the affected-set lens.

Tool calls: none. The manifest's context held every definition traced.
