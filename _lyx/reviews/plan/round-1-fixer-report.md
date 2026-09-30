# Fixer report — plan round 1

Every finding in `round-1-review.md` is fixed in the plan; `lyx loom validate-plan` exits 0 afterwards, with informational `create-new-unit` findings only.
The overview's `approved:` key is untouched.

## Changes

- **F1 (BLOCKING).**
  Card 1: `Shed.stepLocked`'s budget arm applies to a counted Stuck only, with the reason skipping the count alone is not enough; a new test covers an exempt Stuck after exactly N counted Stucks routing to `OnStuck`, then blocking on the next counted one.
  Card 10: the shedengine-driven done-wait test seeds N counted Stucks (budget spent) instead of N-1, and a matching awaiting case is added.
- **F2 (MEDIUM).**
  Card 5 targets `wiring_commitstatus_integration_test.go` and adds real-pair tests: a friction note and a drive report land in the weft HEAD, and a friction archive rename commits both the addition and the deletion.
  Card 7 creates `commitrecords_integration_test.go`: an uncommitted drive report is committed and pushed, and a second call on a clean tree adds no commit.
  Card 16 states that it stubs the commit link and points at those tests; `Uses` entries added for the accessors the new tests read.
- **F3 (MEDIUM).**
  Card 4 became `04-driver-liveness-prereqs.md` (the old `04-loomengine-driver-strand-name.md` removed): alongside the strand-name relocation it declares `plan:internal/reedengine#ErrNoSession`, wraps it in `Engine.requireSessionLocked`'s no-session return only, and tests it in `nosession_integration_test.go` over `newColdScratchEngine`.
  The sentinel went into card 4, not a card of its own, because a new card before card 9 would renumber twelve cards and every cross-reference, and card 4 already exists solely to give batten's liveness probe what it needs.
  Card 9: `DriverAlive` maps `ErrNoSession` to not live, and its body is factored into `driverAliveFrom` with a table test of its answers.
  Overview: Card Index entry 4 renamed, the `liveness-error-is-live` Shared Decision covers the absent-session case and applies to cards 4, 9, 10, and `./internal/reedengine/...` joins both `go test` verify lines.
- **F4 (MEDIUM).**
  Card 10's docs list covers the extra `battenshed/doc.go` statements (report path, never-liveness sentence) and `docs/overview.md` statements (halted-child restart, 12-hour uncommitted watch), and it targets `internal/battencli#maxStatusHistoryEntries` for its stale comment.
  Card 15 gains `docs/overview.md` as `Prosa`: the teardown refusal sentence and the archive tag.
- **F5 (LOW).**
  Card 10: the done-seen marker is removed on Done and on any non-done Call; marker removal failures are hard errors; new tests for both, and the past-grace warning is asserted via `logger.SetOutput`.
- **F6 (LOW).**
  Card 13 targets `internal/fabricengine#CleanupResult`, which gains a once-per-verb `ArchiveSkippedReason`, with a no-origin test; its ImpactSummary is updated to match.
- **F7 (LOW).**
  Card 14's test uses a new untracked file inside a not-yet-tracked drive-reports directory.
- **F8 (LOW).**
  Card 1's ImpactSummary names both `episodeStuckCount` callers, and the intent records that batten's status envelope already bounds rendered history.
- **F9 (NIT).**
  Card 10's intent states the `start.go` premise that the resume rests on.

## Deferred

None.
