# Decision record — the orch resume pointer arrives twice after a compact cycle

## Goal

After a compaction, the watcher's pointer step skips the pointer whenever the session-start hook delivered it, so the session receives it once.

## Scope

- In: the pointer step's delivery check in `internal/orchengine/watcher.go`, the state it reads in `internal/orchengine/state.go`, the package doc's description of that check, and the watcher tests that pin it.
- Out: the hook verb, `orchengine.ResumeContext` and the mark file's format; the text match stays.

## Decisions

### D1 — A mark counts from the baseline before the boundary the reload follows

A reload records, in a new persisted State field, the time a hook delivery for its pointer may start: the compaction baseline as it stood before the reload moved it to the new boundary.
The pointer step counts a mark as the hook's delivery when its time is at or after that field and its text equals the pending pointer.
Both compaction paths, the requested compact cycle and the automatic compaction read at a turn end, record it;
the reload after `/clear` records the baseline unchanged, which keeps its current behavior.

Rationale: the hook runs before Claude Code writes the `compact_boundary` record, so the boundary time is later than the hook's mark;
the previous baseline is the newest boundary already handled, so it precedes this compaction's hook and follows every earlier compaction's hook.
The hook fires only after a compaction, and every return to idle clears the mark, so a stale mark with the same text can only come from an earlier compaction, whose hook ran before the previous baseline.
A requested cycle alone has a phase-entry time, so one rule for both paths takes the baseline.

Bound: a state file an older binary wrote has the field zero, and a zero field accepts any mark whose text matches, which the cleared mark and the text match still guard.

### D2 — The field is persisted and cleared with the reload

The field lives in State, so a watcher restarted mid-reload reads the same lower bound, and is zeroed by every return to idle and by a fresh launch.

Rationale: the pointer step can run again after a restart, and an in-memory value would be lost there.

## Constraints

- `PATTERN-tolerant-records`: the new State field decodes as zero from an older state file and is written through `internal/state`.
- `PATTERN-test-speed`: tests use the watcher's injected clock and run in parallel.

## Auto-mode assumptions

- The hook's clock and the transcript's clock are the same host's wall clock, as the existing check already assumes.

## Open risks

- A compaction whose hook runs before the previous boundary's record is written is not possible, since compactions of one session are sequential.

## Acceptance criteria

- D1: a mark stamped before the compaction boundary but after the previous baseline, holding the pending pointer, skips the typed pointer after an automatic compaction.
- D1: the same holds after a requested compact cycle.
- D1: a mark older than the previous baseline, or holding other text, still types the pointer.
- D2: the field is zero again after the reload returns to idle.
