# Refusal audit: webster dispositions, shed goto, refusal way-forward table

All 31 batches reported `done` and are committed; this session resumed with every batch already done, so it ran only the integration stage.
The plan-level verify command ran once in an integration fork and reported `status: OK` at head `19c4be05a`, with no deviations.

The plan's cards, by area:

- **Webster audit and dispositions (01–05, 07–11, 14–16).**
  Audit findings carry stable keys and a correctness/policy class.
  A state.json ledger dispositions each finding once per run.
  Each batch records its card set.
  Rebaseline accepts an on-disk plan edit.
  A batch-fail primitive archives the report.
  `record-batch` warns on policy findings and fails the batch on correctness findings.
  Unattributable reports are archived and the batch is re-driven.
  Post-batch failures fail the batch, and drift about later cards only warns.
  `recover-batch` proceeds from a failed batch and hands the strand its failure digest.
  `summary.md` gets an Audit warnings section.
  Run exit warns, passes dispositioned findings, or demotes done to stuck.
  Regression tests pin the 2026-09-30 wedges.
- **Webster CLI and Master (06, 12, 13).**
  `lyx webster rebaseline` is exposed.
  Failed batches and archived reports are persisted and committed before the error envelope.
  The Master stencil ladder gained the `batch_failed` and `report_archived` rungs.
- **Shed goto (17–22).**
  The history-only goto outcome and its segment budget boundary are declared.
  The status-file mutation behind goto and the generic verb body are in.
  goto is registered in the shed, loom and batten subtrees.
  The missing-producer and exhausted-budget stops name goto, and loom accepts goto history and names it from Loom-Preflight's half-finished stop.
- **Refusal sweeps (23–27).**
  Refusals in the webster engine, webster CLI, shed, loom CLI and loom gate now carry a way forward, each with tests that reach it.
- **Docs, skill and invariant (28–31).**
  The refusal table contract doc, the package docs, the ly-drive skill lookup, and the Refusal Way-Forward Invariant in `CONSTRAINTS.md` (with goto in the Shed Verb-Set list) are written.

The status envelope lists one extra terminal `stuck` record (number -1, slug `unknown`) from an earlier attempt; it was not a batch in this card list and needed no action.
