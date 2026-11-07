## Goal

```markdown
The operator wants tests faster as a priority (2026-11-07).
Items 7 and 9 of `docs/research/speed/tier1-test-revision-2026-11-06/report.md`, read before starting.

## Seen

- Tests wait real time: `internal/gitexec`'s `TestRun_RemoteBounds` waits real deadlines and a literal `sleep 2` (3.5 s of the package); `internal/gatecli`'s `TestGateTest_DetachedRun` has stand-in delays of 2 to 5 s and leases of 3 s; `internal/termwindow` has a 3 s row; `internal/verifyrun` has 1 s expiries.
- Tests repeat builds every pass: `TestBuildInto_UnwritableDirReturnsError` builds a binary only to fail writing it (2.3 s); `TestCrossCompileLinux` and the dotnet fixture build cost about 5 s of `cmd/lym`.

## Wanted

- Take real time out of each wait through an injected clock, or by waiting on the event itself instead of a fixed sleep; never by a shorter bound.
  A bound that covers a process or binary start keeps its length: on OSL-1046 Cortex XDR holds each new binary about 10 s on its first start, and those bounds were widened because a held binary read as a failure (operator, 2026-11-07).
  Check each wait's history in git before changing it, and leave alone any wait whose only fix would be a shorter bound.
- Have `TestBuildInto_UnwritableDirReturnsError` fail before building, and keep `TestCrossCompileLinux` and the dotnet build in the whole pass while a card or scoped gate runs them only when a file they guard changes, such as a `_linux.go` file.
- No test loses what it pins.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - armdeadline-package-var-clock (class consistency, severity MEDIUM, open in rounds 1)
  - card-watch-skip-untested (class design, severity MEDIUM, open in rounds 1)
  - guard-step-watch-drop-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
