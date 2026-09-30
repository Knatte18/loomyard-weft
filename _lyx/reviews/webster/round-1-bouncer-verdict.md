---
verdict: BLOCKING
rationale: "F1-F7 are fixed and verified (build, vet, untagged and integration tests pass), but three semantic-line-break violations remain in new or changed lines: recordedVerdict's doc comment (run.go:558, ', or it kept'), approvalActedContent's doc comment (innerrun.go:83, ', so identity comparison'), and the changed docs/overview.md lines 430 and 434 (', and every self-bounce', ', so their own reason', ', so a step-driven lifecycle')"
---

# Webster-Review round 1 — Bouncer verdict

Review range: `product.parent` is `main` (from `_lyx/shed/awaiting-status-history-growth/status.json`), merge base `724b78b50`, eleven commits: the four card commits and the fixer's seven one-finding commits.

## What holds

The diff implements all four cards as the plan states them.

- `appendOrFold`/`foldsInto` match only on outcome, exemption, `Producer`, `Output` and the `GateAttempts` pointee, ignore the last entry's own `Repeats`/`LastAt`, and copy before touching `next[n-1]`, so the step-1 slice is never written through.
- The budget arm still reads `st.History` through `episodeStuckCount`, which skips exempt Stucks, so a fold cannot move a counted boundary.
- `recordedVerdict` classifies every `persist`/`persistTransient` caller in `stepLocked` correctly under the held run lock: appends lengthen, folds keep the length and bump `Repeats`, and the pause writes, the step-3b resume write and the no-outcome skip arm compare equal and carry `activity.last` forward exactly as the old `appended` test did.
- `callAwaiting` decides newness by identity alone, records `len(status.History)` of the status this Call read, keeps the re-approve text for a longer history or an old-layout marker, and the `ErrChildNotParked` arm still returns before the marker write.
- Every test bullet the cards list has a named test.

Per-card mechanical check for the Edit groups (cards 1, 2, 4): the blast-radius pass covers every target, not only the first.
`HistoryEntry` construction outside `shedengine` is limited to three empty-slice seeds (`loomshed/seed.go`, `battencli/arm.go` twice); the readers outside it read lengths, ranges or counted Stucks; every `persist`/`persistTransient` call site is in `stepLocked`; `approvalActedFile`/`approvalActedFileSuffix` have the one caller in `callAwaiting`; `maxStatusHistoryEntries`'s value and `recentHistory` are unchanged.
The Create groups add nothing equivalent to an existing symbol.

## Round-1 findings

All seven are resolved.

- **F1** (`fa0c632f2`): the skip arm now copies onto `make([]HistoryEntry, 0, len(st.History))`, so an empty history stays `[]`, and `TestStep_NoOutcomeErrorKeepsEmptyHistoryAnArray` pins it by decoding the raw file.
  The fix leaves `recordedVerdict`'s answer for the skip arm unchanged (equal length, equal `Repeats`).
- **F2–F6**: the named Go comment lines now break at their sentence and independent-clause boundaries.
- **F7**: the two named boundaries in `docs/overview.md` are broken.

The report carries no `## Focus departures` section, so there is nothing to ratify or reject.

## What still blocks

CONSTRAINTS.md's Comment Line-Break Convention governs new and changed Go comments, and CLAUDE.md's markdown rule governs the changed `.md` lines; both break at an independent clause after a comma and a coordinating conjunction.
Three sites in new or changed lines still join such a clause.

1. `internal/shedengine/run.go:558`, `recordedVerdict`'s new doc comment: "the history got longer, or it kept its length while ..." keeps the independent clause after ", or" on one line.
2. `internal/battenshed/innerrun.go:83`, `approvalActedContent`'s new doc comment: "..., never inside it, so identity comparison alone decides ..." keeps the independent clause after ", so" on one line.
3. `docs/overview.md`, the lines this diff changed:
   line 430 keeps ", and every self-bounce repeats the same ... pair" on one line;
   line 434 keeps "(the other rows carry no `on_stuck`, so their own reason is the persisted `error`)" and ", so a `step`-driven lifecycle reports it ..." on one line.
   F7 broke two boundaries on these same lines and left these three.

Each is a soft-break at the named boundary, with no untouched line rewrapped.
The legacy `write; it selects ...` line in `persist`'s doc comment is not listed: its only change was dropping a trailing fragment, and breaking at its semicolon would mean rewrapping the untouched line above it, which the convention forbids.
