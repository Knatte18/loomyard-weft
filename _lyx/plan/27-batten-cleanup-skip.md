# Card 27 — batten-cleanup-skip

**Edit:**
- `internal/battencli#judgeStatus` -> `func judgeStatus(data []byte, prefix, slug string) (candidate, tracked bool, reason string)`
- `internal/battencli#judgeRunDirectory`
- `internal/battencli#TestCleanup`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#TestCancelIntegration_BlockedRunWithNoPair`

**Intent:**
D16: `lyx batten cleanup`'s skip of a run that is not done names `lyx batten cancel <slug>` as its way forward, since issue #724's blocked-run half is met by the cancel.
`judgeStatus` takes the slug and words the skip `<prefix>run is <state>; way forward: "lyx batten cancel <slug>" ends it, and a later cleanup removes it`; `judgeRunDirectory`, its only caller, passes the slug at both calls.
A run read only from the records HEAD, its directory deleted on disk, would otherwise meet the cancel's nothing-left refusal, which counts the records only on disk; so `hubCancelDeps`' `RecordsPresent` also answers true when the prime's records HEAD still tracks the run directory, and the cancel's `RemoveRecords` then commits that deletion, so the named `lyx batten cancel <slug>` progresses and a later cleanup finds nothing to skip.
`TestCancelIntegration_BlockedRunWithNoPair` gains a row: a blocked run whose directory is committed and then deleted on disk, with no pair or claim, is cancelled, its deletion committed.
`TestCleanup`'s not-done rows assert the way forward names `lyx batten cancel r1` and so on per slug, for a run read from disk and one read from the records HEAD.

**ImpactSummary:** One skip reason gains its way forward; judgeStatus's one caller is on this card.
