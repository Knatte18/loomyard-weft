# Arbiter fixes: relaunch, verify halt, stencils, scope-guard deviations

Both batches recorded `done`, with no reported deviations from the declared file-ops.

## Batch 01 (cards 1–4), head `58e6b3200`

- Relaunch skips its goto when Admit reports the target already in effect.
- The arbiter and tackler consumes admit a verdict already in effect.
- The arbiter refuses a Merge-In or Publish verify halt whose own record names a target.
- `lyx loom peers` gives each run's embedded-recipe row and the caller's own.

Informational scope warnings named `haltAtVerifyRow`, `verdictInEffect` and `loomRowNumber` as touched outside the batch's own target glyphs.

## Batch 05 (cards 5–8), head `3b66dbe74`

- The alignment and arbiter stencils name the `lyx board get` JSON payload.
- The alignment agent settles landing order itself.
- The scope guard names each finding's file and reports uncovered symbol-less files; the `internal/planglyph` test budget rose accordingly.
- `record-batch` fills the digest's deviations from the scope guard alone, and `Report.Deviations` stays as an ignored field.
- The implementer stencil asks for `status` and `head_sha` only, and the Master stencil reads the guard's list from the digest.

Informational scope warnings named `filesWithSymbols` and `targetsCover` in `internal/planglyph/scope.go`.
