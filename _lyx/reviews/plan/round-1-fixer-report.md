# Plan-Review round 1 — fixer report

Every finding in `round-1-review.md` is fixed.
`lyx loom validate-plan` passes over the edited plan (`ok: true`, informational `create-new-unit` findings only).
The overview's `approved:` key is untouched.

## Changes

- **F1 (MEDIUM), exit hook placement.**
  Card 5 now creates `cmd/lyx/exitsweep.go` (`notifyExitAndSweep`, `exitSweepBounds`) and `cmd/lyx/exitsweep_test.go` in place of `internal/loggerconfig/exit.go` and `exit_test.go`.
  The Intent states why cliwire is ruled out, and why `cmd/lyx` root is the remaining D4 site.
  Card 2 gains `plan:internal/loggerconfig#ConfigPath` and a test for it, so the Warn names the file without the unexported module name; card 5's Uses swap `internal/configengine#ConfigFile` for it.
  The overview's prose and third Shared Decision, card 5's `logger.go` wording, and its README wording now name `cmd/lyx`'s exit hook.
- **F2 (MEDIUM), reconcile seeding untested.**
  Card 3 targets `internal/configsync/configsync_test.go` with a test that `ReconcileAll(dir, true)` seeds `logger.yaml` equal to the template and reports `Applied: true`.
  It uses `internal/configsync#ReconcileAll`.
  `internal/configsync` has no build-tagged tests, so the verify section is unchanged.
- **F3 (LOW), card 3 ImpactSummary.**
  It now names the fabric clone reach point (seed and commit on the weft primary, `hubforge` hubs), and concludes that no pinned list needs an edit beyond `TestNames`.
- **F4 (LOW), never-armed absent-config case.**
  Card 5's `lyx --help` integration case now runs with an absent and with an invalid `logger.yaml`, and says why the absent run matters.
- **F5 (LOW), card 4 redirects.**
  Card 4 states the `SetDurableSinkDir(t.TempDir())` redirect for every case and why the `go test` gate needs it; the `NotifyExit(1)` case also asserts `Redirected` and `Dir`.
- **F6 (NIT), stale comment.**
  Card 5 rewords `isLyxWorktree`'s doc comment in `sink.go` to name `cmd/lyx`'s exit hook calling `logger.NotifyExit(code)`.

Card 5 also pins the seeded integration traces' mtimes to within the last 14 days, so the `trace_retention_count: 0` case's "every seeded trace survives" holds under the age bound.

## Deferred

None.
