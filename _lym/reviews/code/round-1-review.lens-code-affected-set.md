```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, listing what the diff's changes force outside its lines.

**Per change, the set found**

1. Shipped text of `lym-batching` and `lym-plan-edits-before-gate` is now bullets under a `##` heading (`shipped.json:5,13`).
   - Pinning tests: `store_test.go:170-195` (`ShippedEntriesArePinned`) is updated, and no other test or constant quotes the old text. `batchingSentence` has no other reference. Covered.
   - Render tests: `render_test.go:72-88` (`claudeFileWant`) and `:213-215` build the expected file from `shippedStore`, so they follow the new text. Covered.
   - `heddlecli/cli_integration_test.go:229` checks slugs only. Covered.
   - Stencil: `contracts/stencils/webster/webster-body-implementer.md:54` carries its own wording of the slice rule ("never one file in slices over several turns"). It does not quote the shipped sentence, and the Goal covers only text quoting the old sentences. Not a finding.
   - Nudge: `batchingnudge.go:101` (`readSliceNudge`) restates the Read-tool override in its own words. It is a runtime nudge, not a quotation, and its meaning still matches. Covered.
   - Docs: `docs/overview.md:276` and `pattern/PATTERN-heddle-instructions.md` describe the layers and name no text or heading. Covered.
   - The `docs/research/turn-audit-2026-10-28/report.md` quote of an older sentence is a measurement report, left as written. The summary says so, and the rubric's measurement-report exception supports it. Covered.
2. Heading level: `render.go:227,244` and `shipped.go:21-27` are unchanged. `shippedHeading` is level one, and entries now open at level two, so there is no skip. `doc.go` is updated to say so. Covered.
3. A lym override replaces the whole entry text, heading included (`shipped.go:76-79`). An override tuned before this change loses the heading, and no code or doc forces that. This follows from the summary's design decision, not a defect.
4. Declared PATTERN changes: the directive declares none. `PATTERN-heddle-instructions` and `PATTERN-lym-instructions-in-code` are still satisfied, since lym's instructions stay in the shipped layer. Covered.
5. Commit message: there is one commit and no `Pattern-Departure:` trailer. None is needed, since no PATTERN entry forbids the change. Covered.
6. Stencil wording, comparing the shipped text against the old paragraph. The meaning survives except for one point:
   - Not-a-finding: the override bullet "Follow the rule above over the Read tool's own advice" sits before the two "Never" bullets. "The rule above" is only the preceding read-whole bullet, whereas the old text made the override cover the never-grep and never-slice rules as well. This is a fine-grained wording difference, and the intent is carried by the adjacent bullets.

**Findings**

- LOW, `consistency`, `internal/heddleengine/shipped.json:5` (the `lym-batching` text). The override bullet "Follow the rule above over the Read tool's own advice to read only the part you need" is a weakened carry-over. "The rule above" resolves to one bullet, and the old sentence tied the override to the whole whole-read, no-slice-after-grep and no-slices-over-turns rule. Rephrase so the override names its rule (for example "Read a file whole… even though the Read tool advises reading only the part you need"). It is borderline, so MEDIUM is not warranted.
- No other finding. Nothing outside the diff quotes the old sentences, and every pinning test is updated.

**Tool calls**
1. Grep for the old and new sentence fragments across the repo (excluding `.git`, `.lym` and `_lym`): to find every quoting file, stencil or doc beyond the manifest's greps.
2. Grep for the slugs and "shipped entr" in `.md`, `.yaml` and `.json`: to find docs and fixtures naming the entries.
