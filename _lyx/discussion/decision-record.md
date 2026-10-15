# Decision record — orch-color-on-reexec

## Goal

Stop the orch watcher from retyping the strand's color into an already colored orch session each time it re-execs onto a new `lyx` binary.

## Scope

- `internal/orchengine`: the watcher's start color (`initCursor`, `typeStartColor`, `typeColor`) and the persisted `State`.
- The start color gets an `orch: injection typed` log line like the reload steps.
- Docs: the color paragraphs in `internal/orchengine/doc.go`.
- Out of scope: the reload sequence's color step, which a `/clear` or compaction needs and which stays as it is.

## Decisions

- **Persist what the color was last typed for.**
  `State` gains the strand and the session's `LaunchedAt` the color was last typed for, written on every successful color typing, the start color and the reload color step alike.
  Rationale: a re-exec keeps the strand and `LaunchedAt`, and every new session gets a new strand guid and a new `LaunchedAt` through `ResetForFreshLaunch`.
- **`initCursor` sets the start color pending only when the persisted pair differs from the bound strand's.**
  Rationale: a re-exec on the same session then types nothing, and a new or adopted session still gets its color.
  Bound: it skips only a color already typed for that same strand and launch; a failed or held typing records nothing, so it is retried as before.
- **The start color logs `orch: injection typed` with step `start-color`.**
  Rationale: the late colors were invisible in the watcher log.

## Constraints

- `PATTERN-tolerant-records`: the new `State` fields are plain JSON keys written through `internal/state`; an older state file without them reads as never typed, so the first watcher on the new binary types the color once.
- `PATTERN-orch-pane-single-writer`: the watcher stays the only writer of the color.
- Nothing is typed unless the idle probe passed on the same tick.

## Auto-mode assumptions

- `LaunchedAt` is the session start the board entry means; a zero `LaunchedAt` from an older state file still compares equal across re-execs.
- The color survives a re-exec of the watcher, since the session itself is untouched.

## Open risks

- A color reset by something other than a new session, `/clear` or a compaction is not retyped by a re-exec any more.

## Acceptance criteria

- A watcher started afresh on a state whose recorded color pair matches the bound strand and launch types no color.
- A watcher bound to a state whose strand or `LaunchedAt` differs from the recorded pair types the color on its first idle tick, once.
- A held or failed start color records nothing and is retried as before.
- The reload color step records the pair, so a re-exec after a reload types no start color.
- The start color writes an `orch: injection typed` line.
