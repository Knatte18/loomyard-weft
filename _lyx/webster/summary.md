# Creel stage 1: per-hub message daemon, standalone

All three batches reached `status: done` on the first `record-batch`, with no recovery and no warnings.

## Batch 01: agentname-program-senders (cards 1 to 6)

Head `a4a5f4d569f0e1baa3b1b93cc8f8bd38ecf4cb84`.
Built program-sender names and EXTERNAL in `agentname`, replaced-binary detection in `selfexec`, parent-pid reading, the pane pid in reed status, and verified send and guarded interrupt on a strand with no run record in `shuttleengine`.
Added the `creelengine` geometry, JSONL journal, fold and retention.
The fork chose the journal record shape, since the plan left it open.
Deviations listed by the scope guard: files in `internal/agentname`, `internal/creelengine`, `internal/selfexec` and `internal/shuttleengine` (`readiness.go`, `run.go`, `run_test.go`, `rundir.go`).

## Batch 07: creelengine-pool-verbs (cards 7 to 11)

Head `100b570192c5450edb6c1205e9e07a757e5d8a12`.
Built the delivery seam, pool and verbs with privilege rules, sender identity from peer ancestry and claims, reminders, injects, holds, pause and the removed-holder sweep, the daemon with socket protocol and Go client, and `PATTERN-creel-store` with its import scan.
The fork reports departures from the cards: the `read` verb follows card 7's wording, not the board design's "oldest unread plus same sender"; `Options` gained `PollClock` and `Ready`; the protocol gained `wait-reply` and `wait-inject`; some card 9 timing details and card 10 protocol shapes differ.
It also reports one edit-rule slip: a function appended to `client.go` through a shell heredoc.
Deviations listed by the scope guard: files under `internal/creelengine` and `internal/testkit/creelfake`.

## Batch 12: hubgeom-creel-geometry (cards 12 to 16)

Head `6f681c49660d736b1b8e988306a8120cd68a35e2`.
Built creel's geometry in `hubgeom`, the Delivery seam over reed and shuttle, the `lyx creel` subtree with daemon start and sandbox scenario, the tmux-tier and llm-tier smoke tests, and the creel design in `doc.go` and the overview.
Open items the fork reported:
- The smoke tests are compiled and vetted only; `TestSmokeCreelFakeProviderPane` and `TestSmokeCreelClaudeStrand` have never run.
- The `creel-core` board entry's body could not be cut to a pointer, because `lyx board upsert` refused an entry held by this run. The orch or the landing has to cut it.
- A friction note is at `_lyx/loom/friction/12-hubgeom-creel-geometry.md`.
Deviations listed by the scope guard: files under `internal/creelcli`.

## Scope guard

Every batch's deviation list is informational and held only files within the three areas above.
The plan-level verify has not been run by this session; the gate runs it next.
