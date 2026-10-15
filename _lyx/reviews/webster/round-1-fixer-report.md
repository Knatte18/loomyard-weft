# Fixer-report, round 1

## Changes

- F1: fixed differently, not as proposed.
  The proposed change, calling `fabricengine.WeftWarpSlug` from `internal/reportengine`, was made and then reverted: `TestEnforcement_FabricVocabulary` bars the bare identifier `WeftWarpSlug` outside the fabric owner set, in code and in comments.
  `pairSlug` stays, with its original comments, and reads the suffix off `fabricengine.RecordsBranchName`, which carries no banned token.
  Resolving the duplication would need `internal/reportengine` to join the owner set, which is outside this round's write surface (deferred below).
- F2: the loop in `bucketRuns` writes the key only when absent, under a one-line comment saying a held phase with no time in the bucket stays present as 0.
- F3: `timeRun` truncates the snapshot and `opts.Sleeps` to whole seconds; `runWaits` truncates the parent-review request and verdict times and `change.At`; `runAttempts` truncates `spawned_at`.
  One `TestComputeTiming` row pins a sleep with fractional seconds: phases plus sleep equal `span_seconds`.

`lyx gate test ./internal/reportengine ./internal/lyxcwd` is green.

## Disputed

None.

## Deferred

- F1, the duplication itself: removing `pairSlug` needs the fabric-vocabulary owner set in `internal/lyxcwd` and `internal/fabricvocab` to admit `internal/reportengine`, or `internal/fabricengine` to export a parse under a name without the banned token.
  Both leave the write surface this round names.
