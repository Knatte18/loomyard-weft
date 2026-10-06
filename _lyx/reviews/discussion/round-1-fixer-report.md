# Round 1 fixer report

All eight findings are fixed in `_lyx/discussion/decision-record.md`; F8 also edits `_lyx/discussion/support-log.md`.

## Changed

- F1: D4 now reads the diff at the start of Finalize, before the catch-up merge-in, so the base is the real fork point; the notice carries merge-base and tip SHAs; D4 states that a completed non-squash landing resumes with an empty diff and queues nothing.
  R2 is restated as an accepted loss for that case; the Notes for the plan writer name the read position.
- F2: D6 states that a local weft branch alone makes the pair live and archive tags are consulted only without one; the non-live paragraph is narrowed to match; the archive-refusal criterion gains its no-local-weft precondition and a local-weft criterion is added.
- F3: D6 states that the unarchived-weft refusal is retired and bounds the live-pair cases; R3 now says the warp branch returns with the weft and names `lyx fabric remove` as the way to archive again.
- F4: D5 states that `add` passes its skip options (leaving today's fork behavior), that `reconcile` and `checkout` always consult origin, and that a repo with no `origin` remote skips step 2; the Bound names the skip opt-out.
- F5: D7 gains a Bound for the archive-guard bypass and says why it never decides the same slug as D6.
- F6: D6 states that a fast-forwarded local weft branch is not rewound on rollback.
- F7: the reconcile `.weft/` criterion gains the Detail and unchanged-action assertions; a criterion for a failed Finalize diff read is added; the fast-forward criterion gains the failed-Add case.
- F8: the added section became `D9 — Plan test targets by file path`; the Webster hold, the sync and re-validation step and the rationale moved to the support log's Interview as item 7.

## Deferred

None.
