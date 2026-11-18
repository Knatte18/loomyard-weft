```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F1–F11).

**Findings**

- LOW — `internal/shedadapters/bouncer_amendment_test.go`, row `AmendmentGenerationRendersNoReviewInputNote` (F3's second proposed row).
  - The row passes without pinning that the note is `none` because the record is an amendment.
  - It carries no review input: `withAmendConfig(2, 0, nil)` sets none, so `generation.inputPaths()` is empty.
  - `ReviewInputNote` (`internal/shedadapters/amendment.go:161`) returns `noNote` on `!found || generation.Amendment || len(paths) == 0`.
  - Dropping the `generation.Amendment ||` clause would leave this row green through the `len(paths) == 0` branch.
  - Fix: give the amendment record a review input, for example with `withReviewInput(path, time.Hour)`, so that only the `Amendment` clause yields `none`.
  - That input would be named in the generation record only if an amendment ever recorded one. If `amendmentInputRow` shows an amendment records the input, reuse that setup and assert `ReviewInputNote == noNote` there.
  - Otherwise use the legacy record shape of `RecordWithOnlyReviewInputReadsAsThatOnePath`, which is an amendment holding a path, and assert `ReviewInputNote == noNote`.

- No other proposal broke on a case I could build:
  - F7: the covering rows exist at `internal/landingshed/prreview_test.go:461` and `:482` and `internal/landingshed/prreview_wait_test.go:824`.
    The deleted test's two-call sequence (a dirty halt, then the reroute on the next call over the same record) is asserted only as separate single-call rows.
    That loses nothing the reviewer did not already name.
  - F2: the existence check covers the exact name `pr-review-send-back-<n>.md`, and `n` is unique per send-back.
    An agent's own note of that exact name would be mistaken for ours, which is implausible.
    A missing friction directory takes the `fs.ErrNotExist` branch and falls through to `friction.NotePath` as before.

**Calls**
1. `grep` for the three covering row names under `internal/landingshed`: to confirm they exist, since the F7 deletion rests on them and my context held only the review's claim.
