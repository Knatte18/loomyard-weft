# Card 14 — a resume line refused by another live pause claims the session into it

**Edit:**
- `internal/tenderengine#sendResumeLine` -> `func sendResumeLine(deps Deps, held pauserecord.Session) (reason string, claimed bool)`
- `internal/tenderengine#liftEntries`
- `internal/tenderengine#TestResume`
- `internal/tenderengine#fakePair`
- `internal/tenderengine#fakePair.Send`
- `internal/tenderengine/doc.go`

**Uses:**
- `plan:internal/pauserecord#ClaimSession`
- `plan:internal/shuttleengine#PausedError`
- `internal/tenderengine#newPair`
- `internal/tenderengine#newEnv`
- `internal/tenderengine#env.seedLive`
- `internal/tenderengine#env.record`
- `internal/tenderengine#hubEntry`
- `internal/tenderengine#recorded`

**Intent:**
D2 for `lyx tender resume`'s resume line.
`liftEntries` already hands a session another live entry covers to that entry under the record lock, so a refusal reaches `sendResumeLine` only for an entry added after that lock was released.
On a send error that is a `*shuttleengine.PausedError`, `sendResumeLine` calls `pauserecord.ClaimSession(deps.RecordPath, pe.Slug, held, deps.Now())` and reports `claimed` true with no reason when the claim recorded the session or found it already owned, so the later resume of that entry gives it its line.
An uncovered answer means the refusing entry was lifted since the send, so the line is sent once more and that send's outcome is reported as any send's is.
A second `PausedError` there means a newer live entry covers the session, so `sendResumeLine` calls `pauserecord.ClaimSession` once more with that error's `Slug`: a claim that records the session or finds it owned reports `claimed` true, an uncovered second claim reports the reason `the send was held by a pause that has since ended and a second one`, and a failed one the claim-failed reason, with no further send, so no send loops.
A claim error reports `the send was held by a pause and the claim failed: <error>`.
`sendResumeLine` finds the session's pair among `deps.Pairs` by slug and, for the empty slug of a prime session such as one creel's interrupt claims (card 15), takes `deps.Prime`, so a prime session a pause recorded gets its line instead of `the pair is gone`.
`liftEntries` lists a claimed session under the report's `Moved`, the sessions handed to another live entry that get no line, and every other answer as today.
A session moved this way is the same cost the resume already accepts: one carry-on line after the later resume.
`doc.go`'s resume paragraph names the claim.

`fakePair` gains an `onSend func(guid string) error` field that `Send` calls first, its non-nil error being the send's answer.
`TestResume` gains a row: a hub entry owning a `p1` session is resumed, and the pair's `onSend` adds a live entry of scope `p1` through `pauserecord.Update` and answers a `*shuttleengine.PausedError` for slug `p1`, the entry a pause added after the lift's lock; the report lists the session under `Moved` and not under `Lines` or `NoLine`, and the live `p1` entry owns the session.
A row answers the refusal once with no live entry left, and the second send lands: the report lists the session under `Lines`.
A row whose `onSend` answers the refusal once with no live entry left, then adds a live `p1` entry through `pauserecord.Update` and answers the second send with a `*shuttleengine.PausedError` for slug `p1`, lists the session under `Moved`, and that entry records it.
A row whose `onSend` answers the refusal twice with no live entry left names the session under `NoLine` with the ended-pause reason.
A row whose refusing entry already owns the session reports it under `Moved` and leaves one copy in the entry.
A row whose record is made undecodable inside `onSend` names the session under `NoLine` with the claim-failed reason.
A row resumes a hub entry owning a prime session (empty slug) over deps whose `Prime` is a fake pair holding it, and the prime pair gets the line.

**ImpactSummary:** one private helper re-signed with its one caller in the same package; the resume report's shape is unchanged.
