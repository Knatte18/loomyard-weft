```yaml
lens: fix-affected-set
outcome: reported
```

Checked the review's set-shaped findings, F1, F2 and F4, by re-running each one's search over `_lyx/plan/`.

Unchecked: none. F3 is `decision`-class and the fixer did not touch it; F5 and F6 are single sites, not sets.

**F4** (sites that name `gate_halt`): fully covered. No `gate_halt` without `_at` and no "both fields" is left anywhere in `_lyx/plan/`.

**F2** (sites that clear or rewrite the gate mark): fully covered. Step 7 (lines 56–63), step 10 and the test paragraph name only `gate_halt_at`, and every one keeps it set until the gate has passed or was not due.

**F1** (the `*_at` fields and the "at or after" sites): steps 3 and 7 and the test row are covered. One finding:

- MEDIUM. Changed text: `_lyx/plan/02-drift-records.md:39`, "Every `*_at` field is a string formatted `Deps.Now().UTC().Format(time.RFC3339)`".
  Summary: the fix widened the format rule from the review's "`*_at` fields the history test reads" to every `*_at` field. Several of those fields are written by CLI verbs that have no `Deps.Now`, and the sentence's placement leaves it unclear whether `CaptureRecord`'s times are covered too.
  Evidence: `recorded_at` is written by `RecordScope` (card 7, line 54), `awaited_at` by `lyx drift defer` (card 7, line 67), `waived_at` by `Waive` (card 7, line 70) and `Acceptance.at` by `lyx drift accept`. A grep of `07-drift-verbs-cli.md` for `Now`, `time.` and `clock` finds nothing, so card 7 gives these verbs no `Deps.Now`, and as written the sentence names a step they cannot carry out.
  `CaptureRecord`'s `slot_acquired_at`, `command_started_at` and `ended_at` (card 2, line 43; card 5, lines 53–55) come right after the sentence. Card 5 states no format for them.
  The fix the review asked for needs the format only on `halt_named_at` and `gate_halt_at`, the two the history test reads. They are written by the producer, which has `Deps.Now`.

Tool calls:
1. Grepped `_lyx/plan/*.md` for `*_at` fields and the "at or after" comparisons, to find every member of F1's set beyond cards 2 and 11.
2. Grepped cards 3, 5, 7, 9 and 10 for a clock seam, and the plan for leftover `gate_halt` and "both fields", to check which writers have `Deps.Now` and F4's set (the second grep also matched files outside this worktree; ignored).
