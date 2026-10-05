# Card 12 — orch-smoke

**Edit:**
- `internal/orchcli/smoke_live_test.go`
- `internal/orchcli/smoke_cycle_test.go`
- `internal/orchengine/doc.go`

**Create:**
- `internal/orchcli/smoke_adopt_large_test.go`

**Intent:**
Cover the live behaviour with `smoke`-tagged tests (D17, D18, the compact cycle), keeping PATTERN-tmux-test-isolation through the package's existing `TestMain`.

- `smokeOrchConfig` takes a cycle mode; the existing soft-cycle and full-cycle smoke tests pass `clear`, since they exercise the four-phase machine.
- D17: the bypass smoke test asserts the bypass-permissions footer in the orch pane's capture instead of running a tool the user's settings may already allow; the subagent half that proves forks run unprompted stays if it does not depend on the user's settings.
- Compact cycle: a new smoke test in `smoke_live_test.go` starts the orch in compact mode with a soft threshold low enough to fire, starts a background task in the session (a background shell that sleeps and then writes a marker) before the cycle, and asserts after the cycle that the session id and the strand are unchanged, the reading is known and below the soft threshold, `cycle_count` is 1, and the background task's completion still reaches the session.
- D18: `smoke_adopt_large_test.go` is opt-in through an environment variable naming a session id, and skips when it is unset.
  It copies that session's transcript into the test's sandbox Claude project directory and adopts the copy, asserting startup within `startup_timeout_s` and no dialog left on the pane.
  Bound: it reads only a copy and never touches the original session or its registry entry.

The implementer runs the large-session adopt test once against a real hub transcript and records the result (transcript size, startup time, outcome) under `internal/orchengine/doc.go`'s Open risks.

**ImpactSummary:** Changes the orch smoke fixtures' config shape and the bypass assertion, adds two smoke tests and records one measured result in the orchengine doc; no production code changes.
