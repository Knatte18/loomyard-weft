# Card 22 — beam-land-request

**Create:**
- `internal/beamcli_test#TestLand_RunsFromATaskPair`

**Edit:**
- `internal/beamcli#landDeps`
- `internal/beamcli#land` -> `func land(deps landDeps, slug string) (map[string]any, error)`
- `internal/beamcli#judgeLandEntry`
- `internal/beamcli#hubLandDeps`
- `internal/beamcli#landCmd`
- `internal/beamcli#landWorld`
- `internal/beamcli#newLandWorld`
- `internal/beamcli#landWorld.deps`
- `internal/beamcli#TestLand`
- `internal/beamcli/doc.go`
- `docs/overview.md`

**Intent:**
D10: `lyx beam land <slug>` runs anywhere in the hub and writes a land request naming the pawl run in the beam worktree's `.lyx/beam/`, which Beam-Await consumes; today's separate landing run goes.

- `land` no longer refuses the acting worktree being the beam itself, since the request is consumed by the beam's own run and the pair is removed only at the run's teardown; `landCmd` loses its acting-worktree argument path and its help describes the request.
- `landCmd` runs anywhere in the hub, a task pair included: it resolves the cwd through `lyxcwd` and the hub's prime through `fabricengine.PrimeName` and `lyxcwd.ResolveWorktree`, never through `runVerb`, whose `ResolveBattenPrime` refuses a worktree that is neither the prime nor a beam.
- It refuses, in order, each naming its way forward: an entry that is not an open beam project, as today (card 1's task wording); a beam with no pair here, as today; a legacy beam whose pair carries the mark and which has no pawl run, no batten status file, naming `lyx beam create <slug>`; a beam whose pawl run is not running at Beam-Await, read from the beam pair's own child status (`shedrun.StatusFile(beam, shedrun.SelfRunID)`), naming `lyx batten status <slug>`; a beam this machine does not own, naming `lyx adopt <slug>`; a beam whose parent another machine owns, naming `lyx adopt <parent subject>` (`lyx adopt root` for the prime); then `BeamLandRefusal`, its open-member and member-pair blockers naming their ways forward.
- On passing, it writes `pawlengine.WriteLandRequest(hubgeom.PawlPaths(beam), LandRequest{RunID: <the child seed's started_at>, RequestedAt: now})` and answers the request's path and `lyx batten status <slug>` to watch the landing.
- `landDeps` loses `ParentWorktree` and `StartLanding`, so `StartFirstPass` loses that caller, and gains `Machine`, `ParentOf`, `ChildStatus(slug)`, `RunID(slug)`, `WriteRequest(slug, req)` and `Now`; `hubLandDeps` fills them over the prime.

`TestLand` rows reach every refusal above with its way forward and a passing land that writes one request naming the run's id; its acting-worktree row becomes a land from inside the beam that passes.
`TestLand_RunsFromATaskPair`, over a `hubforge` hub, runs `lyx beam land <beam>` from inside a task pair and reaches the verb's own refusal of a beam with no pawl run, never the prime-or-beam refusal.
`beamcli/doc.go`'s land section and `docs/overview.md`'s beam line describe the request.

What it lets through, bounded: the request is only a file; Beam-Await accepts only one naming its own run, writes the landing hold first, stops the dobby and re-runs every check above but the run-at-Beam-Await one before anything lands.

**ImpactSummary:** the land verb's tail changes from starting a run to writing a request, inside beamcli; its removed seams had no caller outside this package.

**Commit:** 22: `lyx beam land` writes a land request the pawl run's Beam-Await consumes
