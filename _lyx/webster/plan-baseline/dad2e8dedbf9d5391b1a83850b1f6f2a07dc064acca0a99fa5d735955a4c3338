---
format: 5
approved: true
language: go
---

# Plan: dobby, the beam daemon that drives batten passes

Give every beam, the hub's prime included as the root beam, a Go daemon, dobby, that starts the beam's released tasks, drives their batten runs in short passes, lands them and cleans up, escalating only to the arbiter and then the nearest live orch.
Batten becomes one pass per `lyx batten run`, its Run-Shed wait one check per pass with its memory in a per-run record; `draft` becomes a board task status; the arbiter joins every shed run's stop chain, batten's included; Finalize's config notice goes key by key; and the orch stops starting runs.

## Card Index

1 — agentname-scoped-program — form and parse the beam-scoped program sender
2 — scoped-program-notice — creel clients and orch notices signed with a scoped program
3 — runproc-dobby-role — the dobby role and an end that spares named roles
4 — dobby-records — dobbyengine's geometry, state record, pass exit codes and beam.yaml
5 — board-draft-rule — draft for every new task at the board's commit point
6 — board-draft-migration — the one-time draft migration, its key, replay and README mark
7 — draft-writers — cancel and claim settlement write draft; the start guard refuses a draft
8 — shed-yield — the Yield outcome that persists nothing
9 — step-envelope-yield — a yielded step does not continue the loop
10 — driver-yield-exit — the Go driver ends a pass on a yield with nothing decided
11 — idlewatch-snapshot — save and restore the parent lock's idle watch
12 — episodeset-snapshot — save and restore a failure-episode set
13 — innerrun-short-check — InnerRun's one check per pass with a per-run memory record
14 — innerrun-spawn-hold-yield — the spawn hold yields with its reason
15 — innerrun-respawn-cap — cap the respawn of a driverless running child
16 — innerrun-api-error-revive — revive an API-error stall before noticing it
17 — land-drops-ready — drop the ready snapshot and the landed notice
18 — row-notices-after-chain — Land and Teardown failure notices after the stop chain
19 — shed-arbiter-config — the arbiter keys in shed.yaml with their resolver
20 — module-arbiter-overrides — loom, darn and batten override the arbiter keys
21 — arbiter-key-copy — copy loom's hub arbiter keys into darn.yaml
22 — shedarbiter-package — the arbiter's call, prompt and consumption in shed
23 — loomcli-arbiter-delegates — loom and darn use shedarbiter
24 — arbiterstop-batten-rule — batten's arbiter eligibility
25 — batten-arbiter-wiring — batten's driver wires the arbiter
26 — batten-pass-exit-codes — exit codes for owned-elsewhere and covered refusals
27 — dobby-start — start every dobby this machine owns
28 — batten-first-pass — retire --window and start a detached first pass
29 — origin-owed-marker — mark a resume judged without origin
30 — batten-continue-pass — the continue pass and the no-reopen full pass
31 — batten-pass-wiring — InnerRun's pass seams from the scheduling dobby
32 — dobby-loop — dobby's lock, owner exit, wakeups and re-exec
33 — dobby-passes — schedule continue and full passes and read their exits
34 — dobby-owned-elsewhere — pause, notice and re-judge a run owned elsewhere
35 — dobby-ready-starts — start ready tasks with max_runs and backoff
36 — dobby-board-tidy — the board-tidy agent every N landings
37 — beam-verbs — lyx beam start, lyx beam dobby, and a dobby from birth
38 — orch-start-dobbies — lyx orch start starts the dobbies first
39 — adopt-refusals — adopt refuses an orphaned run and ends with the dobby start
40 — teardown-dobby-liveness — a dobby-driven run reads live; reed down spares dobby
41 — finalize-key-notice — Finalize's config notice key by key
42 — orch-role-stencil — the orch role for dobby

## Shared Decisions

### Decision: dobby's packages and import edges

- **Decision:** the daemon, its records, its start helper and its exit codes live in `internal/dobbyengine` (told geometry); its hub constructors `DobbyGeometry` and `DobbyStart` live in `internal/hubgeom`; the verbs and the daemon's seam wiring live in `internal/beamcli`.
- **Rationale:** `battencli` imports `orchcli`, and `beamcli` imports `battencli`, so the start every one of `orchcli`, `battencli`, `adoptcli` and `beamcli` calls must sit below all four; `battencli` reads the parent dobby's state record and dobby reads batten's pass exit codes, so the codes sit in the package `battencli` imports.
- **Applies to:** cards 4, 26, 27, 28, 30, 31, 32 to 39

### Decision: the arbiter's shed-layer package

- **Decision:** the arbiter's call, prompt, spec, verdict gate and resume consumption move to a new `internal/shedarbiter`; its records stay in `loomengine` and its settings resolver moves to `sheddriver` beside the tackler's; the stencil keeps its name `loom-template-arbiter` and gains a `{{.recipe}}` marker.
- **Rationale:** `sheddriver` may not import `loomengine`, where the records live with their other readers (reports, checklist, decisions); a shed-layer package beside it can, and batten imports it without linking tree-sitter.
- **Applies to:** cards 19 to 25

### Decision: the records-stop line of a successful landing

- **Decision:** with the `landed` notice dropped, the Land row posts one line only when the prime's records side did not pull.
- **Rationale:** the decision record's Added-after-Discussion entry "A records stop on a successful landing still notices the orch" (parent, 2026-10-25): the line is not the dropped `landed` notice but a failure that leaves the prime's board behind and needs judgement, like D9's kept Land failure notices.
- **Applies to:** cards 17, 42

### Decision: a running task shares two files

- **Decision:** running task `discussion-scope-in-lenses` also edits the orch role stencil and renames `Merge-In` to `Parent-Align` in the landing texts `arbiterstop` and `sheddriver` hold; whichever lands second takes the other's wording in those files.
- **Rationale:** the decision record's constraint.
- **Applies to:** cards 18, 24, 25, 42

## PATTERN changes

- `PATTERN-dobby-starts-runs` — adds: Starting or driving a board task's run: its parent's dobby, from an empty status, in batten passes; a hand `lyx batten run` is one pass.
- `PATTERN-owner-machine` — changes: a continue pass dobby schedules into a run it already drives is not a resume and needs no origin verdict; the first pass of a run after a dobby start or a machine wake, every pass while a full pass is owed, and every hand pass, is a resume and gets one.
- `PATTERN-shed-stop-chain` — adds: A shed run's stop: rule table, tackler, arbiter, then parent, for every recipe; arbiter keys in shed.yaml with module overrides.
