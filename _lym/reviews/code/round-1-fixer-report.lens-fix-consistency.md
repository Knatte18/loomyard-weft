```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for consistency with its surroundings (lens fix-consistency).

**Findings**

1. LOW — `tools/godocreflow/main.go:12-13` (package doc), against the new `layoutParagraph` behavior in `split.go`.
   - The doc says "A line whose text reaches the rules' length limit is split at such a break; a line with no such break stays whole, and the run names it for a hand rewrite."
   - Since F1, a paragraph whose layout would leave a changed line at the limit also stays whole, even when a break exists.
   - Example: the "only break leaves a piece over the limit" row. The line has an accepted colon break but the paragraph stays as written.
   - The summary.md "What changed" text was updated for this case, but the package doc was not.
   - A reader of the doc would expect a partial split in that case.

2. LOW — `_lym/landing/summary.md`, Evidence paragraph.
   - It says the 606 named lines are ones "no accepted or written break can shorten".
   - After the F2 change, the report covers every prose line that reaches the limit in an eligible group's final text.
   - That includes lines in paragraphs kept as written under F1, where a partial break exists but would leave a piece still at the limit.
   - It also includes unchanged legacy long lines. The sentence "they are already over the limit as written" holds, but the "no break can shorten" clause is looser than what the report counts.

3. LOW — `tools/godocreflow/split.go`, `splitLongLine` doc: "or the earliest when none does".
   - This is still accurate for the function itself.
   - It is now only reached when the paragraph is discarded, or when the earliest-cut piece is a written line. So the doc describes a path whose result the caller usually drops.
   - The doc does not mislead, but `layoutParagraph`'s "A line with no such break stays whole" can read as contradicting it. Optional tidy-up.

**Checked, consistent**
- `collectEdits` has one caller outside the tests, `main.go:62`, and it is updated.
- The test helper `reflowSource` is updated.
- `edit.longLines` and the `slices`/`strings` imports are removed with no leftover users in the files I saw.
- `docs/benchmarks/test-suite-prune-rest.md` only lists the package's coverage numbers, a measurement report that is allowed to go stale.
- No other doc references the removed stderr wording.

**Calls made**
1. `grep` for `godocreflow` and `collectEdits(` across `.go`, `.md` and `.yaml`, excluding `tools/godocreflow`: to find twins and callers outside the changed package, which my context did not hold. The first, broader grep was too noisy to use.
