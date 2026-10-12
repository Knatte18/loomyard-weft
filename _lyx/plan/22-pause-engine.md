# Card 22 — pause-engine

**Create:**
- `internal/pauseengine/doc.go`
- `plan:internal/pauseengine#SessionInfo` -> `type SessionInfo struct`
- `plan:internal/pauseengine#Pair` -> `type Pair interface`
- `plan:internal/pauseengine#Deps` -> `type Deps struct`
- `plan:internal/pauseengine#PauseRequest` -> `type PauseRequest struct`
- `plan:internal/pauseengine#PauseReport` -> `type PauseReport struct`
- `plan:internal/pauseengine#Pause` -> `func Pause(deps Deps, req PauseRequest) (PauseReport, error)`
- `plan:internal/pauseengine#ResumeRequest` -> `type ResumeRequest struct`
- `plan:internal/pauseengine#ResumeReport` -> `type ResumeReport struct`
- `plan:internal/pauseengine#Resume` -> `func Resume(deps Deps, req ResumeRequest) (ResumeReport, error)`
- `plan:internal/pauseengine#StatusReport` -> `type StatusReport struct`
- `plan:internal/pauseengine#Status` -> `func Status(deps Deps) (StatusReport, error)`
- `plan:internal/pauseengine#holdReport` -> `type holdReport struct`
- `plan:internal/pauseengine#readHolds` -> `func readHolds(deps Deps, rec pauserecord.Record) (holdReport, error)`
- `internal/pauseengine/pause_test.go`
- `internal/pauseengine/resume_test.go`
- `internal/pauseengine/testmain_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the module engine behind `lyx pause` and `lyx resume`: the pause and resume sequences of D3 and D6 and the status read, over seams, with every path told (`PATTERN-told-geometry`).

Seams: `Pair` is one hub pair as the sequences see it: `Slug()`, `Sessions()` (its live shuttle-run strands as `SessionInfo{Strand, GUID, Slug}`, the loom driver included), `Busy(guid)`, `Live(guid)`, `Interrupt(guid)` (shuttle's `Runner.Interrupt`), `Send(guid, text)` (shuttle's guarded paste-submit) and `Running()` (the run's current producer and whether it reads running).
`Deps` holds `RecordPath`, `Pairs`, `NetOps` (`pauserecord.ListNetOps`), `Locks` (`parentlock.Store.List`), `OrchWatcherLive`, `Caller` (the caller's strand name, empty for `EXTERNAL`), `Now` and `Sleep`. The CLI fills them (card 23).

`Pause` (D3), scope `pauserecord.ScopeHub` for an empty slug:
1. Under the record lock it adds the entry with `PausedAt`, `GraceEnd` = now + grace and the caller; a live entry of the same scope adds none and the steps run for sessions busy now.
2. Grace: it polls each second until every covered session reads idle and no covered network operation is in flight, or `GraceEnd` passes, or the entry is no longer live, which ends the sequence with `ResumedDuringGrace` and no Esc.
3. Interrupt, per covered session that reads busy, skipping the caller's own strand (reported skipped) and a session another live entry already owns: under the record lock it checks the entry is still live and that no live entry owns the session, records it in the entry, releases the lock, then calls `Interrupt`; it reads `Busy` once more and interrupts once more when still busy. An entry gone at the check stops the step there, so no Esc goes after a resume. A pane-check refusal (`shuttleengine.ErrPaneNotReady`) is reported as refused.
4. Report: interrupted, already idle, still busy after the retry, refused, skipped; network operations in flight, naming those under a parent lock; each parent branch a covered run holds with the runs queued behind it, and each covered run waiting in a queue with the runs behind it (`readHolds` over `Locks`); covered runs reading running with no live session, as running a Go producer, with the producer; and under a hub entry the orch's case: the orch ran the pause, delegated to a live orch watcher, or not interrupted because no watcher is live. The engine never touches the orch.
Covered sessions are those of every pair the entry covers. Bound: Esc only, never text, only to sessions shuttle tracks that no live entry owns; it kills no pane, strand or process.

`Resume` (D6), an empty slug resuming every live entry: a slug with no live entry of its own refuses, naming `lyx resume`, when a live hub entry covers it, and is a no-op success otherwise; resuming nothing is a no-op success.
Under the record lock it moves each resumed entry to the resumed list with `ResumedAt`, and moves each session it owns to another live entry still covering that session's pair, with no line. After the lock, for each remaining non-orch session: a strand not live, a session reading busy, and a guarded send refused with `shuttleengine.ErrSessionBusy` (a draft in the box) each get no line and are named with the reason; otherwise `pauserecord.ResumeLine` goes once through `Send`. The orch's session is left to the watcher (card 21) and reported as such.
Bound: at most one line per session per resume, only into a session the entry owns; a session recorded by a pause killed before its Esc gets one surplus line, which its text tells it to end on.

`Status` reads the live entries, the same network operations list as `PauseReport` (from `deps.NetOps`, marking each one run under a parent lock) and the same holds report, changing nothing.

Tests, over a fake `Pair`, a fake clock driving `Sleep`, and a temp record:
- `pause_test.go`: the entry is on disk before the first `Interrupt` and the session is recorded before its Esc; the grace waits for busy sessions and network operations and never past `GraceEnd`; a busy session is interrupted at most twice and reported still busy; the caller's strand is skipped; nothing running writes the entry with empty lists; a repeated pause adds no entry and interrupts only sessions busy now; an entry resumed during the grace sends no Esc and the report says so; a slug pause under a live hub entry owns only unowned sessions; a pane refusal is reported; the report names network operations (marking the parent-lock one), a held branch with its queued run, a covered waiter with the run behind it, a Go producer, and each orch case; a pause and a resume racing in two goroutines leave a decodable record and no `Interrupt` after the entry's resume.
- `resume_test.go`: one line per owned session; none into a dead, busy, idle-at-pause or draft-holding session, each named with its reason; a session recorded by a killed pause gets one line; with hub and slug entries overlapping a session is interrupted once, resuming the first entry moves it with no line, and the line goes when the last resumes; `lyx resume <slug>` under only a hub entry refuses naming `lyx resume`; resuming nothing succeeds with nothing resumed; `Status` reports the live entries and holds, lists an open network marker under a parent lock marked as such, and writes nothing.
The budget gains the package's row.

**ImpactSummary:** New engine package with no caller until card 23.
