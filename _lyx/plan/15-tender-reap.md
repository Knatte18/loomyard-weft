# Card 15 — lyx tender reap ends the orphans of one fresh reading

**Create:**
- `internal/tenderengine#ReapReport`
- `internal/tenderengine#Reap`
- `internal/tenderengine#TestReap`
- `internal/tendercli#reapCmd`
- `internal/tendercli#refuseReapCaller`

**Edit:**
- `internal/tendercli#TestPauseAndResume_ApplyTheCallerRules`
- `internal/tenderengine#Deps`
- `internal/tendercli#Command`
- `internal/tendercli#HubDeps`
- `internal/tenderengine/doc.go`
- `internal/tendercli/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/tenderengine#readProcesses`
- `internal/tenderengine#ProcessReport`
- `internal/runproc#End`
- `internal/runproc#EndProcess`
- `internal/runproc#SurvivorsError`
- `internal/runproc#Ended`
- `internal/tendercli#callerOf`

**Intent:**
D8's verb: once a pair is removed no `remove` or `reed down` target is left, so one verb ends what survived, and status names it.
It is the only new verb (auto-mode assumption).

`Deps` gains `EndOwner func(owner string) ([]runproc.Ended, error)` and `EndProcess func(pid int, startTime string) ([]runproc.Ended, error)`.
`Reap` takes one fresh reading through `readProcesses` and ends exactly its orphans: each orphan owner whose every record reads orphan through `EndOwner`, one call per owner; each orphan record of an owner card 14 kept a record of, as working inside a registered worktree, through `EndProcess` with its recorded start time; and each unregistered orphan through `EndProcess` with the start time of that same reading, so a pid reused since is left alone.
`ReapReport` carries `Ended` and `Survivors` (`ended`, `survivors`, empty lists by default) and the process part it read.
Survivors come from each `*runproc.SurvivorsError`; with any, `Reap` returns the report and an error naming each survivor's pid, and its role when registered, and the way forward of running `lyx tender reap` again once it is gone; any other seam error is returned after the remaining orphans are tried.
Bound: `Reap` never ends a `HubOwner` process, a process whose pair exists, one whose working directory lies inside a registered worktree, or one of a slug whose batten reads running, because `readProcesses` lists none of those as orphans; those are ended by the pair's end verbs or by `batten-cancel-verb`.

`reapCmd` is `lyx tender reap`, no arguments, operator audience, with a help text naming what it ends and its bound.
It takes `lyx tender pause`'s caller check with no slug: `refuseReapCaller` lets the orch and an external caller through and refuses an agent with the way forward of asking its parent session, which runs `lyx tender reap`; the refusal runs before any hub read.
The envelope carries the report under `reap`, and an error exits non-zero with the report beside it.
`Command` adds it; `HubDeps` fills `EndOwner` with `runproc.End(RunProcDir(hub), owner, os.Getpid())` and `EndProcess` with `runproc.EndProcess(pid, startTime, "")`.

`TestReap` rows over fake seams: an orphan owner is ended and a pair-backed owner is not; an owner with one record kept for its working directory has only its other records ended, through `EndProcess`; a `HubOwner` record is never ended; an unregistered orphan is ended with the reading's start time; a survivor yields the error naming its pid and role, and the way forward of running `lyx tender reap` again once it is gone, and the other orphans are still ended; an `EndOwner` error that is not a survivor error is returned after the remaining orphans are ended; nothing to reap returns an empty report.
`TestPauseAndResume_ApplyTheCallerRules` gains `reap` in its agent loop, asserting the refusal names asking the parent session, and a `reap` call in its orch section, asserting no caller refusal.
The docs, the overview's tender module-table row (`docs/overview.md`, the `internal/tendercli/` line listing the verbs) and its tender line, and `Command`'s short and long text listing the verbs, name `reap`, its caller rule and its bound.

**ImpactSummary:** one new verb and engine sequence; two nil-safe Deps seams; no existing verb changes.

**Commit:** 15: lyx tender reap ends orphaned run processes
