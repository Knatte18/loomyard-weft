---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/loomcli/arbiter.go:settleFresh
    summary: after Start succeeds, a failed attempt.json rewrite or marker write returns an error and leaves the live arbiter unstopped and, for the marker, unheld by batten
    fix: in settleFresh, when the second WriteArbiterAttempt or WriteArbiterMarker fails after Shuttle.Start, call run.Stop() before returning the error, mirroring stopRunAndHandOn's rule that the run must not proceed beside a live arbiter; add a row to the arbiter verb's table-driven test with a failing marker write asserting the stop.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/loomcli/circling.go:9-12
    summary: the package comment has a stray leading "// //" and says arbiter eligibility admits only a stop on a Bouncer escalation, which arbiterEligibility does not (it also admits blocked and error stops)
    fix: replace the line starting "// // That is bounded" with "// That is bounded by which stencils name the verbs, the Bouncer escalation brief and the arbiter stencil only;", and reword the next line to say the circling verbs refuse unless the run awaits at a Bouncer row with a pending escalation, instead of attributing that to the arbiter's eligibility.
  - id: F3
    severity: NIT
    class: consistency
    location: internal/loomengine/template.yaml:35
    summary: the unrelated parent_review_wait_min line lost its two-space gap before the inline comment, unlike every sibling line
    fix: restore two spaces between "60" and "#" on the parent_review_wait_min line.
---

### [LOW] Live arbiter left unstopped when a post-spawn record write fails

**Location:** `internal/loomcli/arbiter.go` (`settleFresh`)
**Issue:** Focus asked whether every failure path falls back to hand-on without losing a stop.
Every wait, timeout, died and verdict failure does, and `attempt.json` is written before `Shuttle.Start`, so every spawn counts against `arbiter_max_spawns`.
The exception is the pair of record writes after `Start` returns.
A failure there returns a plain error with the arbiter live, so the driver falls back to hand-on but nothing ends the strand, and a failed marker write also leaves batten's hold unengaged.
The stop is not lost, so this is LOW.

### [LOW] Circling package comment is garbled and overstates the arbiter bound

**Location:** `internal/loomcli/circling.go:9-12`
**Issue:** The rewritten paragraph starts with "// //", and claims the arbiter's eligibility admits only a stop on a Bouncer escalation.
`arbiterEligibility` admits `blocked`, producer and bootstrap error stops as well, so the sentence is false as written.

### [NIT] Template comment alignment disturbed

**Location:** `internal/loomengine/template.yaml:35`
**Issue:** The diff touches `parent_review_wait_min` only to drop a space before its comment.

## Checked and found sound

- Cards 2, 3 and 12: `shedengine.Relaunch` takes the run lock and one status-lock hold, runs `CheckGotoTarget` before `Admit`, and writes only after `Admit` passes.
  It keeps `pause_requested`, `pause_before` and `pause_after`.
  The consumption hook runs in the `--until-stop` waiter before `runUntilStop`.
  `consumeArbiterAttempt` checks the stop key and pause conditions in `Admit`, marks `applied`, and a refused or repeated attempt returns the error envelope carrying `arbiter_attempt`.
- Cards 9 and 10: eligibility is one pure function over the closed list; budget exhaustion, off, timeout, died, missing, unparseable and precondition paths all end as a hand-on or a refusal naming the hand-back way forward.
- Cards 14 to 16: the hold releases when the marker is absent, the strand row is not live, or the deadline plus `noticeReportWait` has passed; `ReviveStrands` stops dead arbiter strands before reed's `Resume`; `internal/battencli` adds only `internal/loomengine`, and its `orchcli` import predates the range.
- Cards 1, 4, 5, 11 and 13: paths, records and config keys match the Shared Decisions, and `--by arbiter` requires a caller whose parsed shortname, slug and role are the run's own arbiter.
- `lyx gate test` over the touched packages, `contracts/...` and `cmd/lyx` is green, including the test-budget check; `lyx loom lint-comments` reports ok.
- Test economy: no new test duplicates a covering public-surface test that I found; `TestRelaunch` is left to cover pause preservation as card 12 states.
