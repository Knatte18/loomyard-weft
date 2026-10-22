# Decision record — convergence-darn-landing-commit

## Goal

`lyx report convergence` maps each darn run's archive tag to its own landing commit, instead of mapping every recent darn tag to one older landing and reporting a "claim one landing commit" clash.

## Scope

- The exact join in `internal/reportengine` reads a tag's task commit only from the run's own records history: the first-parent walk below the tag stops at the commit that opened the run's pair.
- A tag with no task-commit trailer in its own history is not exact-attempted and goes to the time join, which already maps a darn run through its landing checkpoint and the board's done mark.
- `internal/fabricengine` exports the subject of the pair-opening commit and its inverse, so the report recognises that commit without restating the text.
- The `internal/reportengine` package doc states the walk's new floor.

Out of scope: making darn runs write a task-commit trailer, and changing how a loom run's trailer is found above its pair-opening commit.

## Decisions

### The trailer walk stops at the run's pair-opening commit

Decision: the walk below a tag ends at the first commit whose subject is the pair-opening subject for the tag's slug, `fabric: record parent branch for <slug>`, without reading that commit's trailer or anything below it.
Rationale: a pair's records branch forks from its parent's, so every commit below the pair-opening commit is the parent's history; the parent's own sync commits carry trailers naming the parent's landings, and a darn run, which writes no trailer of its own, borrowed the newest of them.
The bound only shortens the walk: a tag whose own history holds a trailer reads the same task commit as before, and a history with no pair-opening commit, from a pair made before the commit existed, walks as before.

### The pair-opening subject is a fabricengine export

Decision: `fabricengine` declares `OriginRecordSubject(slug)` and its inverse `OriginRecordSlug(subject)`, and `Add` commits the origin record under `OriginRecordSubject`.
Rationale: the report names no fabric side and restates no record format, so it reads the subject through a neutral export, as it already does for the archive tag and the task-commit trailer.

## Constraints

- `PATTERN-told-geometry`: reportengine derives no path and imports no `lyxcwd`; the change adds no import outside its allowlist.
- `PATTERN-fabric-vocabulary`: reportengine and its tests spell no side name.
- `PATTERN-test-economy`: the regression is a row in the existing landings table test.

## Auto-mode assumptions

- The time join is the intended route for a run with no task-commit trailer of its own; the board entry's "as loom runs' tags do" asks for an own-landing mapping, not for the exact join specifically.
- A landing checkpoint is committed for darn runs as for loom runs, since both land through the same Finalize.

## Open risks

- A darn run whose time window holds no unclaimed parent commit stays unmapped; that is the time join's existing behaviour, not a wrong mapping.

## Acceptance criteria

- Two darn runs, each with no trailer in its own history above its pair-opening commit, over a parent history whose sync commit names an earlier landing, map each to its own landing through the time join, with no clash finding.
- A tag whose trailer sits above its pair-opening commit still maps through the exact join; the existing exact-join rows keep passing.
- `go test` for `internal/reportengine` and `internal/fabricengine` passes through `lyx gate test`.
