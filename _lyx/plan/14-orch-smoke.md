# Card 14 — orch-smoke

**Create:**
- `internal/orchcli/smoke_cycle_test.go`

**Edit:**
- `internal/orchengine/doc.go`

**Uses:**
- `internal/orchcli/start.go`
- `internal/orchcli/watch.go`
- `internal/orchcli/cycle.go`
- `internal/loomcli/smoke_driverstrand_test.go`

**Intent:**
Prove one full cycle against a real Claude session, and record the two open risks the decision record leaves to this test.

`smoke_cycle_test.go` carries the `smoke` build tag and follows the existing smoke tests' conventions for skipping when tmux or the `claude` binary is unavailable and for building a hub with `hubforge`.
From the hub's prime it:

1. Seeds an `orch` config with a low `threshold_tokens` and short idle grace and poll interval.
2. Runs `lyx orch start --no-attach` and waits for the strand to report ready.
3. Sends the session a turn that starts a short background task (a backgrounded `sleep` whose completion notification arrives after the cycle) and waits for that turn to end.
4. Runs `lyx orch cycle`.
5. Polls `lyx orch status` with a bounded attempt count until `cycle_count` is 1 and `phase` is `idle`.

It asserts that a new, non-empty handoff file exists under the handoffs directory and equals `last_handoff`.
It asserts that the pane capture after the cycle shows the resume prompt naming that file and no longer shows the pre-cycle transcript.
It asserts that the session's current transcript, read from the latest Stop payload, differs from the pre-cycle one, which proves the context was cleared.

It then observes, without failing on the answer, whether the background task's completion notification reached the resumed session, and whether the session's `SendMessage` address before and after `/clear` matches (read through `ListAgents` output in the pane or its transcript, whichever the implementation finds workable).
It logs both answers with `t.Log`.
It ends with `lyx orch stop`, and a cleanup removes the strand if the test failed first.

Replace the `## Open risks` entries in `internal/orchengine/doc.go` with the observed outcomes when the implementer can run the smoke test on a machine with a live Claude Code install.
When it cannot run, the entries say so explicitly: unverified, pending a smoke run.

**ImpactSummary:** Adds one smoke-tagged test and edits one doc comment; no production code changes.

**Verify:**
go vet -tags smoke ./internal/orchcli/...
