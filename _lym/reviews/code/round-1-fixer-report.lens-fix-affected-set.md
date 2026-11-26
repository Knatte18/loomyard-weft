```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check, covering whether each set-premised finding (F1, F2, F3, F4, F7) was fixed for every member.

**Findings I left unchecked:** none. F5 and F6 have a single site each, not a set.

**Set results**
- **F1, statements of the bullet rule:** the `continuesItem` doc, the `SectionBullets` doc, the spec and the summary were all updated. The repo-wide search found no other statement of the rule; the `internal/pattern` hits concern a different parser.
- **F3, consumers of the bullet text:** the non-test callers are `raddleparser/check.go`, `loomshed/raddlefile.go` (`SplicePart`), and `CheckDecisionIDs` in `boardcli`, `loomcli` and `loomshed/gates`. The summary's three named readers cover them.
- **F2, F4 and F7:** two LOW findings below.

**Findings**

1. **LOW: new exclusion branches have no test row.**
   - **Changed text:** `htmlBlockStartLine`, `thematicBreakLine` and `atxHeadingLine` in `internal/discussionparser/decisionids.go`, and the new `TestSectionBullets` rows.
   - **Evidence:** the HTML rows are only `<!-- note -->` (condition 2) and `<details>` (condition 6). Removing the alternative for `<(script|pre|style|textarea)`, `<\?`, `<![a-z]` or `<!\[CDATA\[` leaves every row passing. `<pre`, `<script`, `<style` and `<textarea` are not in the condition 6 tag list, so nothing else catches them. The only thematic-break row is `---`, so removing the `\*` or `_` alternative also passes.
   - **Possible change:** extra rows in the same table, for example `<pre>` and `___`.

2. **LOW: spaced `* * *` and `- - -` never reach the new thematic-break check.**
   - **Changed text:** `thematicBreakLine` (its `(-[ \t]*){3,}` and `(\*[ \t]*){3,}` spaced forms) and the doc and spec claim that a thematic break ends a bullet.
   - **Evidence:** `SectionBullets` tests `isBulletLine(line)` before `continuesItem`, and `* * *` starts with `* `. So `- a\n* * *\n- b` should read as `["a", "* *", "b"]`; I traced this from the code and did not run it. Raddle coverage would then require a part to carry the `* *` bullet.
   - **Scope:** this behaviour predates the fix, which did not make it worse. F4's proposed change ("with only spaces between") implies these forms should end the item without producing a bullet. `---`, `***` and `___`, the forms F4 names, work.

**Tool calls**
- `git grep` for the `SectionBullets`, `SplicePart`, `CheckDecisionIDs` and `raddleparser.Check` callers: my context's earlier search did not cover `raddleparser.Check` or the `SplicePart` callers.
- `git grep` for statements of the bullet rule across all tracked files: the earlier search was narrower in wording and predates the fix.
