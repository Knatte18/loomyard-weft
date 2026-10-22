# Card 16 — Hibernate confirmation reads shell-only background work as settled and waits out forks

**Create:**
- `plan:internal/hibernateengine#confirmForkRereads` -> `const confirmForkRereads = 720`

**Edit:**
- `internal/tenderengine#SessionStatus`
- `internal/tenderengine#sessionStatuses`
- `internal/tenderengine#TestStatus`
- `internal/hibernateengine#confirmed` -> `func confirmed(status tenderengine.StatusReport, landLockHeld bool, requester string) (ok bool, hubEntry bool, forkOnly bool, reason string)`
- `internal/hibernateengine#awaitConfirmation` -> `func awaitConfirmation(deps Deps, report tenderengine.PauseReport, requester string) (confirmation, string)`
- `internal/hibernateengine#Watch`
- `internal/hibernateengine#TestWatch`
- `internal/hibernateengine#sessionReading`
- `internal/hibernateengine#fakeWorld`
- `internal/hibernateengine#fakeWorld.ownRequest`
- `internal/hibernateengine/doc.go`
- `internal/tenderengine/doc.go`

**Uses:**
- `internal/hibernateengine#statusReading`
- `internal/hibernateengine#confirmRereads`
- `internal/tenderengine#fakePair`

**Intent:**
D10: a suspend freezes a shell and the wait loop it holds and resumes them on wake, so a session busy only on shells does not withhold a hibernate; a fork is never interrupted and its next batch is already held by the pause, so confirmation lets it finish within a longer bound.

`SessionStatus` gains `Background []string` (`background`): the kind of each outstanding task of a busy state whose cause is background, filled by `sessionStatuses` from the state's `Outstanding`, provider-neutral (`PATTERN-shuttle-provider-seam`).
`TestStatus` gains a row whose fake pair's session reads busy on background work with a shell and a fork and asserts `background` lists both kinds.

`confirmed` counts a session that reads busy with cause background, and whose every `Background` kind is a shell, as settled.
A session with an outstanding fork still withholds, with the reason naming its strand and the fork.
It evaluates every withholding reason of the reading and reports `forkOnly` when each one is fork work, never when a network operation is in flight, the land lock is held or any other session is busy.
When `requester` parses (`agentname.Parse`) as the orch, the session whose strand is the requester is left out of the reading, since the orch is not interrupted by a hub pause and its writes that matter are withheld through the land lock and the in-flight network operations.
`awaitConfirmation` keeps reading every `confirmInterval`: after `confirmRereads` re-reads (ten minutes) it fails unless the latest reading's withholding was `forkOnly`, and after `confirmForkRereads` (sixty minutes) it fails whatever the reasons, both engine constants on the injected sleep, so no test waits.
`Watch` passes the request's `Requester`.
Bound: the land lock and in-flight network operations withhold as before; the longer bound applies only to a reading where forks are the only reason.

`sessionReading` gains a variadic background-kinds argument: with kinds given the session reads busy with cause `background` and those kinds in `Background`, and every existing call keeps its reading.
`fakeWorld.ownRequest` takes the requester from a new `fakeWorld` field, `requesterExternal` when unset, so a row can seed the orch's strand name.
`TestWatch` gains rows through `fakeWorld`'s readings: sessions busy only on shells confirm at the first reading; a session with a fork withholds past ten minutes of fake time and confirms once the fork's reading turns idle before sixty; a fork still outstanding at sixty minutes fails naming it; a fork beside an in-flight network operation fails at ten minutes; an orch requester's own busy session is left out and a non-orch requester's is not.
The docs' confirmation paragraphs state the shell rule, the two bounds and the orch's exclusion.

**ImpactSummary:** confirmed and awaitConfirmation are re-signed, their only callers being awaitConfirmation and Watch; SessionStatus gains one field every other reader ignores.

**Commit:** 16: hibernate confirmation settles shell-only background work and waits out forks
