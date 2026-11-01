```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the diff. It has no Go logic change, so I judged the stencil rendering path and the test case.

**Per-branch list**

| Changed item | Case I ran | Result |
|---|---|---|
| `TestRenderPrompts` fork case, new `contains` entry 1 (`internal/websterengine/template_test.go` ~810) | Matched the asserted string against the stencil text at `webster-body-implementer.md:56`, including the 3-space indent and the backticked `testdata/`. | Correct. The asserted text is a prefix of line 56. |
| New `contains` entry 2 (`template_test.go` ~811) | Matched against `webster-body-implementer.md:57`, including the apostrophe in `hit's`. | Correct. The existing assertion on `` `lym quarry resolve &lt;glyph&gt;...` `` passes with `&lt;` and `&gt;` unescaped, so the render does not HTML-escape. The apostrophe therefore also survives. |
| Renamed case title | Compared the new title with the assertions. | Correct. The two added assertions check sweep prose, so the title matches. |
| Template placeholders | Checked whether the added lines open any `{{` that could break parsing. | Correct. The four added lines contain none. |
| Stencil sentence "after any rename tool run" (`webster-body-implementer.md:55`) | Walked the order: the rename tool runs, then the Rename paragraph greps for zero references to the old name, then the sweep greps all retired names. | Not contradictory. The two greps overlap, and the sweep is the broader one. |
| Stencil sentence "the second sanctioned deviation" (`webster-body-implementer.md:58`) | Compared with line 43 (the guard-step fix is first, the self-fix for a failure the card caused is second) and with line 69 ("the first sanctioned deviation"). | The numbering is consistent, but the wording is not sound (finding F1). |

**Finding F1**
- **Severity:** MEDIUM
- **Class:** consistency
- **Location:** `contracts/stencils/webster/webster-body-implementer.md:58`, against lines 43 and 119-124
- **Summary:** The sweep sentence calls an edit to a hit outside the card's targets "a fix to a failure the card caused". The sweep runs before the card's first edit, so no failure exists yet. The second sanctioned deviation is bounded by the two-pronged test (a test exercising changed code, or a scan flagging a file the card wrote) and by the attempt cap. Neither bound applies to a pre-edit sweep, so the sweep edit has no stated limit.
- **Evidence:**
  - Line 43 defines the second deviation as a fix to a failure under the self-fix section's test and cap.
  - Lines 119-124 define when the card caused a failure, and every condition there needs an observed failing test or scan.
- **Suggested fix:** Reword line 58 to say that an edit to a retired-name hit outside the card's targets is a fix to the stale reference the card's own retirement caused. It should also say the edit stays limited to that retired name's references.

No other findings.

**Tool calls:** none. The manifest already held the diff, the full stencil and the test hunk.
