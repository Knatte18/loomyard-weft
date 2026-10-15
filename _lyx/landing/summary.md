# creel stage 1: per-hub message daemon with delivery through shuttle

Adds `lyx creel`, the one way to reach an agent from a terminal, a Go program or another agent, built standalone.
No other module sends through it yet; the operator tries it from a terminal.
The wiring of the orch watcher, batten, the parent-review gate and drivers (`creel-notices`, `creel-agents`) follows on the operator's word.

## What changes

- New `internal/creelengine`: a per-hub daemon with a message pool folded from an append-only JSONL journal (one file per UTC day, 7-day retention that carries unread mail, queued injects and live holds forward), a socket protocol, a Go client and a delivery scheduler.
  The daemon is hidden (`lyx creel daemon`), started by any verb that finds it down, and exits when its binary is replaced.
- New `internal/creelcli`: the `lyx creel` subtree with `post` (`--reply`, `--wait-reply`, `--timeout`), `inject` (`--interrupt`, `--ttl`, `--wait`, `--timeout`), `interrupt`, `forward`, `read` (`--from`, `--all`, `--wait`, `--timeout`), `inbox`, `log`, `hold` and `release`.
  It also holds the delivery adapter over shuttle, reed's persisted state and `pauserecord`.
- Mail is pulled: `post` stores, and the daemon types one reminder line when the receiver is idle, backing off from 5 minutes to 1 hour after a reminder turn that ends unread.
  Injects are typed ahead of reminders, expire by TTL counted only while the receiver is live and unpaused, and an unlanded inject is journaled as unconfirmed and never retyped.
- `internal/shuttleengine`: idle check, verified send and guarded interrupt now work on a live reed strand with no shuttle run record, so a hand-added `lyx reed add --cmd claude` strand is reachable.
- `internal/agentname`: upper-case program-sender names and `EXTERNAL`.
- Supporting changes: `selfexec` replaced-binary detection, a parent-pid reader in `internal/proc`, the pane pid in reed status, `hubgeom` creel geometry, a `creelfake` test kit, `PATTERN-creel-store` with an import scan, updated PATTERN backgrounds, `docs/overview.md`, sandbox scenarios and test-budget rows.
  The whole creel design, including the parts stages 2 and 3 build, is in `internal/creelengine/doc.go`.

## Design decisions to know

- The socket is `<lyxtemp root>/creel-<shortname>-<hash>.sock`, with a hash of the hub path, not the board's `$XDG_RUNTIME_DIR/lyx/creel-<shortname>.sock`: `XDG_RUNTIME_DIR` is unset on Windows and often under WSL, and the shortname alone collides between two hubs of one repo.
- On Linux the sender is derived from the peer's process ancestry against the strands' pane processes, not from the environment alone.
  A `LYX_STRAND_NAME` that disagrees is refused, and an unreadable state refuses rather than resolving to EXTERNAL.
  Off Linux identity is the environment, so a confused agent can pass as EXTERNAL or claim a program-sender name; on every platform a process detached from its pane can.
  Privileged senders (EXTERNAL, program senders, orch strands) type arbitrary single-line text into any pane at its next idle.
- Program senders are upper case (`ly:BATTEN`), superseding the board's lower-case example.
- Creel never types into an orch strand in stage 1 (`PATTERN-orch-pane-single-writer` is unchanged); mail to an orch is stored, and its reminders are shown as withheld.
- Reminders back off after their own unanswered turn, a departure from the board's "repeats each time the receiver goes idle".
- `read` marks messages read only when the client acks that it wrote them out; a connection that drops without the ack leaves them unread.
- Shuttle's record-less send confirms landing by the pane alone and keeps no pending-send record; `PATTERN-pane-text-injection` gains a bullet for it.
  `engine-registry` touches the same `Send`/`Interrupt` lookup, so whichever lands second adapts.
- Constants in `creelengine` replace a `creel.yaml`: retention, TTLs, backoffs and caps.

## To check by hand

- `TestSmokeCreelFakeProviderPane` (tmux tier) and `TestSmokeCreelClaudeStrand` (llm tier) were compiled and vetted but have never run.
  Run both before relying on end-to-end delivery.
- The `creel-core` board entry's body is still the full design; `lyx board upsert` refused to edit it during the run.
  Cut it to a pointer at `internal/creelengine/doc.go` after landing.
- `read` sets and `inbox` in `internal/creelengine/verbs.go` (`Store.Read`): confirm plain `read` (oldest unread plus every other unread from its sender), `read --from` and `read --all` match the board's sets, and that the `inbox` sequence in the plan-review finding `read-sets-inbox-sequence-ambiguous` reads right.
- Native Windows is unverified: AF_UNIX sockets need Windows 10 1803 or later, and creel is the first socket server in the code.
- The delimiting of subjects in reminder lines lowers but does not remove the prompt-injection risk of sender-written text typed into a pane.
- Reading every worktree's reed state per sweep grows with the hub's pairs; it runs only while unread mail exists.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` were not confirmed by the run records read for this description.
