# Round 2 fixer report

All seven findings are fixed in `_lyx/discussion/decision-record.md`; F2 also edits `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` passes on the result.

## Changed

- F1: D4 now says landing never merges the weft, so the merge-base is always the fork point and every run of Finalize computes the same diff; the non-squash "empty diff" sentence is gone, the Bound is a repeat notice on a resume that reaches Done again, R2 is rewritten as that duplicate risk, and an Acceptance criterion pins the re-queue on resume.
  This reverses the round-1 F1 fix, whose premise (a non-squash resume sees the task branch as an ancestor) held for warp but not for the weft, which `Merge` and `MergeIn` never touch.
- F2: a failed diff read no longer halts the landing; Finalize warns, lands and queues one notice that the diff could not be read.
  D4, A1 and the Acceptance criterion changed; the support log's Q5 answer changed and the Stuck alternative is added to its Rejected alternatives.
- F3: D5's Bound now says only `add` has a skip opt-out, that `reconcile` and `checkout` stop on an unreachable origin with no opt-out, and that this includes `checkout` to a branch with no weft anywhere; A4 covers `checkout`; criteria added for `checkout` with an unreachable origin and for both verbs on a hub with no weft origin.
- F4: D6 states that under `SkipGit` or `SkipPush` no origin is consulted, the pair is live only by a local weft branch, and the warp branch forks from HEAD.
- F5: criteria added for a live pair with no origin warp, for `add` under a skip option with an origin weft, and for `reconcile` adopting an archive-covered origin weft; `.weft/` is asserted on the origin-adopt paths of `add` and `reconcile`.
- F6: the ordering rule moved into D6 (the live-pair decision, the divergence refusal and the origin probes precede the first creation).
- F7: the Notes for the plan writer section is removed; no requirement lived only there once F6 moved.

## Deferred

None.
