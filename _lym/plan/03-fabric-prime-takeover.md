# Card 3 — fabric-prime-takeover

**Edit:**
- `internal/fabricengine#retryWeftPush`
- `internal/fabricengine#isTakeoverStop`
- `internal/fabricengine#IsPushOwed`
- `internal/fabricengine#withoutFinalizeEntered`
- `internal/fabricengine#HandOverRequest`
- `internal/fabricengine#HandOverTips`
- `internal/fabricengine_test#TestRejectedRecordsPush_ReadsTheTakeover`
- `internal/fabricengine_test#TestHandOverTips`
- `internal/fabricengine_test#TestIsPushOwed`

**Extends:**
- `internal/fabricengine/doc.go`

**Create:**
- `internal/fabricengine#TakeoversError`
- `internal/fabricengine#TakeoversError.Error`
- `internal/fabricengine#ErrTakeoverPassOwed`
- `internal/fabricengine#EndedHandedOver`
- `internal/fabricengine#primeTakeovers`
- `internal/fabricengine_test#newPrimeRunsAhead`

**Uses:**
- `internal/fabricengine#takeoverStop`
- `internal/fabricengine#ReadUpstreamTakeover`
- `internal/fabricengine#ReadHandOverMark`
- `internal/fabricengine#replayWeft`
- `internal/fabricengine#EndedSet`
- `internal/fabricengine_test#newDivergedPair`

**Intent:**
Make a rejected push of the prime's records, which carries every batten run's commits and names no one run, read the takeovers (D6).
Today `retryWeftPush` skips the takeover read when `runSlug` is empty; the prime's pusher (card 5) pushes with no slug.
A prime push that names a run reads that one run's takeover alone, so it would replay another taken-over run's commits.

- `primeTakeovers` reads, from the fetched upstream tip, the takeover entry of every batten run that a `Run-Transition` trailer
  of a local commit origin lacks names (`ahead`, oldest first, each slug once), through `ReadUpstreamTakeover`.
  A run whose entry names a machine `here` holds, or that has no entry, is left alone.
  A run taken over whose hand-over mark names the entry's machine is listed in `handedOver`.
  Any other taken-over run is returned in `pending` as its `*TakeoverError`.
- `retryWeftPush`, for a non-zero `here` on a records worktree that is the prime's, runs `primeTakeovers`
  after its fetch and before any replay or lock, whatever `runSlug` is:
  a prime push that names a run, such as batten's cancel and re-adopt record pushes and `pushOwedRunCommits`,
  carries every batten run's commits as the pusher's does.
  A pending takeover returns a `*TakeoversError` holding every pending one, with nothing replayed or pushed.
  A failed read returns an error wrapping `ErrTakeoverPassOwed`, with nothing replayed.
  With no pending one, it merges each `handedOver` slug into the `EndedSet` it passes to `replayWeft` as `EndedHandedOver`.
  `replayWeft` then sets each such run's commits aside under local set-aside tags, as it does an ended run's,
  and carries the other runs' commits in their order:
  a batten commit stages only its own run's records (`battenRunCommitPaths`), so every path of a commit lies under one run's directory.
  A pair's records worktree, alone, keeps `takeoverStop` on its `runSlug` as today.
- `withoutFinalizeEntered` keeps every `EndedHandedOver` entry, since a handed-over run's commits are never replayed here.
- `isTakeoverStop` also reports a `*TakeoversError` and an error wrapping `ErrTakeoverPassOwed`,
  so `pushClassified` returns either as it is, with no failed-push record and no notice, and `rejectionRetryKind` retries neither.
- `IsPushOwed` reads an error wrapping `ErrTakeoverPassOwed` as owed, so the pusher retries the whole pass in its backoff.
  It reads a `*TakeoversError` as not owed: the caller hands those runs over (card 4).
- `HandOverRequest` gains `PrimeRecords *lymcwd.Location`, the prime whose records hold the run's unpushed transitions.
  When it is set, `HandOverTips` pushes the prime's local records tip under an owner archive tag named for the run,
  with the same owner archive tag helpers it uses for a pair's tips, before the hand-over mark is written.
  A failed archive returns before the mark, wrapping `ErrArchiveFailed`, as every failed step does today.

The change lets no push through that was refused before: it only reads more takeovers on a push of the prime's records,
and a failure of that read holds the push owed instead of recording a refusal.

`fabricengine/doc.go` gains a line on the prime's takeover read.

`newPrimeRunsAhead` builds a hub whose prime records branch holds one unpushed commit per slug,
each staged under that run's record directory and carrying its `Run-Transition` trailer,
with origin's prime records branch moved past the local tip by another clone's commit.

`TestRejectedRecordsPush_ReadsTheTakeover` is tier1 and gains prime rows on `newPrimeRunsAhead` with two runs:
- one run taken over by another machine: a `*TakeoversError` naming that run alone, and nothing pushed;
- the same, pushed with a run slug naming the run not taken over: a `*TakeoversError` naming the taken-over run, and nothing pushed;
- the same run with this machine's hand-over mark naming the entry's machine:
  the push lands with the other run's commits in their order, and the handed-over run's commits are absent from origin;
- the same, with the handed-over run's local batten record standing at its Land row, which `withoutFinalizeEntered` reads as entered:
  the handed-over run's commits are still absent from origin;
- origin's tip holding the taken-over run's status file as invalid JSON, which `originStatusAtTip` returns as a parse error:
  an error wrapping `ErrTakeoverPassOwed`, nothing pushed and no failed-push record.

`TestHandOverTips` is tier1 and gains rows with `PrimeRecords` set and `Pair` nil, a batten run whose pair is gone here,
so no pair archive runs before the prime's:
an owner archive tag on origin names the prime's local records tip before the hand-over mark is written;
with a pre-receive hook on origin refusing tags, an error wrapping `ErrArchiveFailed` and no hand-over mark.

`TestIsPushOwed` is tier0 and gains rows: an error wrapping `ErrTakeoverPassOwed` reads owed; a `*TakeoversError` reads not owed.
The file header of `pushowed_test.go` names the pusher's owed retry as the reader of `IsPushOwed`, instead of a transition seam's.

**ImpactSummary:** Changes the prime's rejected-push retry and widens two error predicates; pair pushes keep their path, and IsPushOwed's other callers only gain one more owed class.
