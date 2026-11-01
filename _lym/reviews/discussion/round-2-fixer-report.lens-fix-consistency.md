```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling text they touch (fix-consistency lens).

Findings:

1. **LOW**: The D6 slug bullet now gives each try its own slug, `adopt-fix-<subject>-t<N>`, but D3 was left as it was.
   - D3 still says "red files the same darn again" (decision-record.md, D3 bullet 5), which a reader may take to mean the same slug.
   - The Goal's "files a darn fix task" is neutral and needs no change.
   - Evidence: D6 now says "Each filing therefore has a slug no earlier run of the subject held". D3 is unchanged.

2. **LOW**: The fix added the new D7 state `ended` but did not add its twin entry under Auto-mode assumptions.
   - `paused` and `landed` each have an assumption line explaining why they sit beside the parent's list: "A run under an operator's own pause reports `paused`, a state beside the parent's list (D7)" and "`landed` covers runs whose…".
   - `ended` is a further state outside the parent's list and has no such line.
   - The fixer's own choices have no assumption line either: the never-reset filing serial, `holds.json`, D9's second marker, and create-then-release.

3. **LOW**: The PATTERN changes entry `PATTERN-front-task` was not updated for two new D8 bounds.
   - D8 now says the land action runs only on the subject's owner: "only while this machine is the effective owner of the fronted worktree's subject".
   - D8 also fixes a front task's routing: "its project, kind and slug are fixed, so a board edit cannot move it to another worktree or slug".
   - `PATTERN-adopt-hold` received its twin edit (cancel). `PATTERN-front-task` still reads only "…runs its `on_land` action only from fronttask's closed table."

4. **LOW**: The D5 green-end removal now covers a "released, running" fix task, which reads against the D4 criterion that was left unchanged.
   - The unchanged criterion: "a re-run of `lym adopt <subject>` while the subject's fix task stands released with an empty status leaves that task unheld, and dobby starts it".
   - That same re-run starts a check. If the check goes green, D5 now removes the fix task, or cancels it if dobby already started it.
   - So "dobby starts it" holds only until the green end. The criterion should either say so or be limited to a red check.

5. **LOW**: The D6 failed-removal notice can lead to two checks.
   - The notice names "`lym board remove <slug>`, then a re-run of `lym adopt <subject>`".
   - The unchanged D8 bound says "An entry removed by hand, not through `fronttask`, reads gone and its action runs; for `adopt-check` that is one more check."
   - When the failed step was the cancel, the record stays. The hand removal then spawns `lym adopt check <subject>`, and the named re-run of `lym adopt` starts a second check of the same subject.
   - The D5 green-end notice, which also names `lym board remove <slug>`, has the same doubling.

No further tool calls were made; the context held everything this lens needed.
