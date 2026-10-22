# Card 1 — pauserecord claims one session into the covering entry

**Create:**
- `plan:internal/pauserecord#ClaimSession` -> `func ClaimSession(path, slug string, session Session, now time.Time) (Entry, bool, error)`
- `plan:internal/pauserecord#TestClaimSession` -> `func TestClaimSession(t *testing.T)`

**Edit:**
- `internal/pauserecord/doc.go`
- `internal/orchengine#TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine`

**Uses:**
- `internal/pauserecord#Update`
- `internal/pauserecord#Record.Owner`
- `internal/orchengine#watchEnv`

**Intent:**
D6: add `ClaimSession` to `internal/pauserecord`, the one claim helper D2's tender resume, D7's creel interrupt and D8's webster batch verbs call.
Under the record lock, through `Update`, it finds the oldest live entry covering `slug` with `Record.Covering`.
An uncovered slug records nothing and returns the zero entry and false.
A covered slug returns true and the entry that owns the session after the call: it appends `session` to the covering entry's sessions and returns that entry, unless a live entry already owns `session.GUID` (`Record.Owner`), in which case it records nothing and returns the owning entry, so a caller that re-reads the entry's liveness (D7) reads the entry whose resume gives the session its line.
A session the covering entry already holds is not appended twice.
The record is written only when a session is appended: the mutate returns without changing the record otherwise, so `Update` still rewrites the file but changes no entry.
The caller chooses the session's shape: an orch session is `Session{GUID: <the orch strand's reed GUID>, Orch: true}`, the shape `orchengine`'s `recordOrchInEntry` writes, and the empty slug of a prime session is covered by a hub entry only, as `Entry.covers` already rules.
An undecodable record is the returned error and records nothing.
The package keeps importing only the standard library and `internal/state` (`PATTERN-leaf-packages`; `TestLeafInvariant_AllowlistOnly` pins it).

`TestClaimSession` is table-driven over a temp record: an uncovered slug records nothing and answers false; a slug a slug entry covers records the session in that entry and answers it; a prime session (empty slug) under a slug entry only is not covered, and under a hub entry is recorded in the hub entry; a session another live entry already owns records nothing and answers true with the owning entry, not the oldest covering one; a second claim of the same session leaves one copy; an orch session is recorded with `Orch` true and no strand name; an undecodable record is an error and leaves the file as it was.

A row added to `TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine` pins that an orch claimed this way before the grace end gets no Esc from the watcher and gets its resume line after the resume: the row pauses the hub, calls `ClaimSession` with the orch session `{GUID: "s1", Orch: true}` and the empty slug, advances past the grace end with a busy orch and ticks, asserts no interrupt call, then resumes the hub, ticks and asserts the resume line was posted once.

`doc.go` gains one sentence naming `ClaimSession` and its two no-record cases.

**ImpactSummary:** one new function in a leaf package and its test, one new row in the orch watcher's pause test, a doc sentence; no existing behavior changes.
