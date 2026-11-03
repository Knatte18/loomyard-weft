```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over a diff that changes only a stencil prose file (`contracts/stencils/drift/drift-template-baseline.md`), with no Go code.

**List (function or branch, case run, verdict)**

- **Template markers, all six.** `parent_directive`, `edit_directive`, `subject_reading`, `capture_command`, `scope_note`, `baseline_path` are all still present as `{{.x}}` in the file as the diff leaves it. No marker was added or removed. Rendering against the `markers_test.go:62` list is unchanged, so the template parses and fills the same as before. Correct.
- **Step 2 scope branch, task the suite cannot reach (documentation only).** The reachability paragraph before the list is untouched and sends the agent to `off`. The list's `off` line ("as above") agrees with it. Correct.
- **Step 2 scope branch, pure refactoring.** The agent picks `full`, as before. Correct.
- **Step 2 scope branch, intended change the suite reaches (the cs hub case: report format, six cases).** The agent picks `full` and does not skip. The compare then shows the six cases as drift, and the verify agent accepts each with a reason. Any other raw line that moves in a case still shows as drift. This is the entry's Expected behavior. Correct.
- **Step 2 scope branch, a test made meaningless or nondeterministic.** The agent picks `partial` with one `--skip` per such test. This is consistent with `RecordScope`, which refuses a skip list outside `partial` and a `partial` scope with no skip (`internal/driftengine/doc.go:55`). Correct.
- **Boundary: every test the task makes meaningless or nondeterministic.** `partial` would skip every test, and the capture would write nothing. The stencil does not warn about this, and neither did the old text. This is not a regression from the diff, so no finding.
- **Step 3 single-call rule.** Unchanged: it still says the verb refuses a second call, call it once, and keep the recorded scope on refusal. Its `--scope <off|partial|full>` and `--skip` text still matches the verb. Correct.
- **Step 4 report, scope not yet recorded.** The agent names each expected drifting test, the lines that change and why, and says every other line must stay. This feeds verify's "judge against the baseline report's expectation" (`drift-template-verify.md:29`). The verify stencil's accept and defect rules still agree with the new wording. Correct.
- **Step 4 report, scope already recorded.** The new line "Name each skipped test and why the task makes it meaningless or nondeterministic" assumes the skip list came from the agent's own judgment. A recorded skip list set by the parent could have another reason, for example a slow test. The agent would then write a reason it cannot know. It is a prose edge case with no code effect.
- **Step 4 report, empty skip list.** The line names nothing, so a `full` or `off` report is not forced to invent a skipped test. Correct.

**Findings**

- NIT, class `consistency`, `contracts/stencils/drift/drift-template-baseline.md:52`. The line "Name each skipped test and why the task makes it meaningless or nondeterministic" presumes the agent chose the skips. When the scope was already recorded, the line before it says "write the report against that scope". A parent-recorded skip may have another reason. Evidence: line 52 read against line 53 and the scope_note marker at line 44.

No other findings from this lens.

**Tool calls:** none, because everything needed was in the inherited context.
