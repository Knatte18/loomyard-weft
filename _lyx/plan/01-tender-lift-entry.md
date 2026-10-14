# Card 1 — tender-lift-entry

**Edit:**
- `internal/tenderengine#PauseRequest`
- `internal/tenderengine#Pause`
- `internal/tenderengine#Resume`
- `internal/tenderengine#TestPause`
- `internal/tenderengine#TestResume`
- `internal/tenderengine/doc.go`

**Create:**
- `plan:internal/tenderengine#EntryRef` -> `type EntryRef struct`
- `plan:internal/tenderengine#Lift` -> `func Lift(deps Deps, ref EntryRef) (ResumeReport, error)`
- `plan:internal/tenderengine#liftEntries` -> `func liftEntries(deps Deps, pick func(record *pauserecord.Record) ([]pauserecord.Entry, error)) (ResumeReport, error)`

**Intent:**
Give the pause engine the two hooks the hibernate daemon needs: a lift of one pause entry by its identity, and a callback that tells a pause's caller the entry it just added.

`EntryRef` names one pause entry by `Scope`, `Caller` and `PausedAt`, with JSON tags `scope`, `caller` and `paused_at`, so the hibernate request file can embed it.
`Lift(deps, ref)` lifts the live entry whose scope, caller and paused-at time all equal ref's (`PausedAt` compared with `Equal`), and lifts nothing when no live entry matches, which covers an entry already resumed, a newer entry of the same scope and an entry another caller added.
It runs the same resume sequence `Resume` runs for that one entry: under the record lock the entry moves to the resumed list and each session it owns that another live entry still covers moves to that entry with no line; after the lock each remaining non-orch session gets `pauserecord.ResumeLine` once through the guarded send, and the orch's session is left to its watcher (`OrchLeftToWatcher`).
To keep one body, move `Resume`'s lock body and send loop into `liftEntries(deps, pick)`, where pick selects and removes the target entries from the record under the lock and may refuse; `Resume` passes its slug selection and its existing refusal of a slug only a hub entry covers, unchanged, and `Lift` passes the identity match, which never refuses.
A lift that matches nothing returns a report with empty lists and a nil error, and writes the record unchanged.

`PauseRequest` gains the field `Added func(entry pauserecord.Entry) error`.
When it is set and the pause adds an entry, `Pause` calls it once, right after the record update that added the entry and before the grace's first poll, so no Esc has gone yet.
A scope that already had a live entry adds none and calls no `Added`.
An `Added` error ends `Pause` at once with that error wrapped, before the grace and any Esc; the entry stays live, and the caller holds it from the argument it was given.
Bound: the hook adds no path past the pause's guards, since it runs in the caller's own process and changes nothing the engine reads.

Rows to add, each a reachable case over the existing `fakePair`, `env` and `fakeClock`:
- `TestResume`: a lift by identity of a hub entry owning `recorded("p1", "a")` and an orch session, beside a live `p2` slug entry, lifts the hub entry alone, gives `a` its line (`p1.sends == ["g-a"]`), reports `OrchLeftToWatcher`, and leaves the `p2` entry live.
- `TestResume`: a lift whose ref does not match the live hub entry resumes nothing, sends nothing and leaves that entry live with its sessions, one row looping over two refs: one naming an entry no longer live, while a hub entry with a later `PausedAt` stands, and one with the live entry's scope and time but another caller.
- `TestPause`: `Added` is called exactly once with the added entry, and at that call the record already holds it, no interrupt has been sent and the clock has not moved; a second pause of the same scope calls no `Added`.
- `TestPause`: with a session reading busy (`p.busy["g-a"] = []bool{true}`), so a pause would key it, an `Added` error is returned by `Pause`, no Esc follows and the entry stays live.
The new rows call `Pause` and `Lift` directly with `e.deps()`, so no helper changes.

The package doc's Pause and Resume sections gain one sentence each: the `Added` callback, and the lift of one entry by identity that lifts nothing for an entry no longer live.

**ImpactSummary:** `Resume`'s behaviour is unchanged and its body moves into `liftEntries`; `PauseRequest` gains an optional field every existing caller leaves nil, so `tendercli` needs no edit.

**Commit:** 1: tender: lift one pause entry by identity, report an added entry
