# Round 3 fixer report

Verdict was APPROVED with four non-blocking findings; all four are fixed, each in its own commit.
Every fix is a comment line break only, so no test was added or changed.

## Fixed

- F1 (`4c811259b`): broke the which, whose and so clauses at the comma in `internal/shuttleengine` — `run.go` (`settleLoadTurn`), `doc.go` (skill-loading paragraph), `fakes_test.go` (`skillReed`) and `startup_test.go` (`TestStartup_SkillLoadTurns`).
- F2 (`c58180358`): broke the since clause in the `skillload.go` file header and the which clause on `classifyWindow`.
- F3 (`401cb56ef`): broke the which clause on `TestWatcher_SkillSkipCauses` and the comma-spliced second clause in `internal/orchengine/doc.go`'s reload-sequence paragraph.
- F4 (`c009b9bf1`): broke the which clause on `runnerSession.LoadSkills` in `internal/orchcli/session.go`.

## Verified

`go build ./...` and `go test` over `internal/shuttleengine/...`, `internal/orchengine/...` and `internal/orchcli/...` pass after the fixes.

## Deferred

None.
