# orch: cycle on a fresh reading, compact mode, batten notices

`lyx orch` could cycle a session on a stale context reading: on 2026-10-04 it cycled a 14k-token session on a 354k reading left from before a `/compact`.
This change makes the reading follow compaction, adds a compact cycle as the default, and replaces the orch's own run watchers with notices that batten queues and the orch watcher delivers.

## What changes

- **Context reading.**
  The provider's reader takes the later of the latest main-chain assistant usage entry and the latest compaction-boundary entry, and reports whether the reading came from a boundary and when.
  It reads the transcript from the end in growing chunks, so a typical read touches only the tail.
  Before an automatic cycle fires, the watcher re-reads the context and fires only if the fresh reading still meets the threshold.
  A requested cycle (`lyx orch cycle`) skips the threshold and is not re-read.
  A fresh launch or adopt resets the reading to unknown.
- **`cycle_mode: clear|compact`** in `orch.yaml`, default `compact`.
  A compact cycle persists a compacting phase and types `/compact <focus>`, rendered from the new one-line stencil `orch-template-compact.md`.
  It completes when a compaction boundary at or after the phase entry is read and the idle probe passes, and it writes no handoff file.
  The soft trigger fires without a DEFER handshake.
  A compaction past `handoff_timeout_s` returns to idle with `compaction timed out` and holds automatic triggers for `soft_idle_s`; a requested cycle is not held.
  The clear cycle is unchanged apart from stencil text.
- **`permission_mode` accepts `bypass` only.**
  Any other value, `prompt` included, is a config load error naming the fix, so every `lyx orch` verb refuses before touching a pane.
  A hub on an old `orch.yaml` must edit it before `status` answers again.
- **Batten run notices.**
  The Run-Shed row writes one `[batten notice]` line per episode when the child leaves running, when the driver strand is found dead, or when the child's `status.json` is unchanged for `notice_quiet_min` (default 45).
  Episode state is persisted, so a batten restart does not re-notify.
  Notices go to a queue under the prime's `.lyx/orch/`, only when the prime's orch state records a strand; otherwise they are logged.
  The orch watcher is the only writer into the orch pane: it types queued notices oldest first, one per tick, only in phase idle on an idle-probe pass.
  `PATTERN.md` gains `PATTERN-orch-pane-single-writer` for this.
- **Stencils.**
  The start, resume and adopt stencils tell the orch to rely on driver messages and batten notices and never to start a polling shell or Monitor.
  The handoff stencils no longer ask for a watcher list.
- **Adopt and attach.**
  An adopt is refused for a live pid only when its process start time matches the registry entry's; a positive mismatch proves reuse and lets the adopt through.
  When the start time cannot be read, a live pid still refuses.
  When the idle probe fails because the pane is too short to draw an input box, the watcher sets `State.Stuck`, which `lyx orch status` shows and the next passing probe clears.
  reed's `window-size latest` is unchanged, with the reason recorded in `internal/reedengine/doc.go`.
- **Docs.**
  `internal/orchengine/doc.go`, `internal/orchcli/doc.go`, `internal/battenshed/doc.go`, `internal/shuttleengine/claudeengine/doc.go`, the `orch.yaml` template comments and `PATTERN.md` describe the shipped behavior.

## Design decisions to review

- Transcript and compaction-boundary shapes and the `/compact` command text stay in `internal/shuttleengine/claudeengine`; the shuttle seam carries a provider-neutral reading.
- Notice delivery is at least once: a watcher that dies between typing a notice and removing its file types it again after restart.
  The queue holds one file per notice, capped at 50 by the appender, which drops the oldest.
- Notices are informational; batten's control flow, budgets and halts are unchanged by them.
- The compact cycle assumes a background task started before `/compact` still delivers its completion, as after `/clear`.
  If that is false, a compact cycle loses an in-flight completion with no handshake to catch it; `cycle_mode: clear` restores DEFER.
- The reader depends on the compaction-boundary entry, a Claude Code internal.
  If its shape changes, the reader degrades to the last assistant entry and a compact cycle times out instead of completing; the hold bounds the retries.
- If the Claude session registry records no process start, the PID-reuse check never lets an adopt through, and the refusal message says so.

## To check by hand

- The live smoke tests were not run in this session: the compact cycle (same session id and strand kept, reading below the soft threshold, a pre-compact background task still delivers its completion), the bypass-footer assertion, and the large-session adopt test.
  The large-session test is opt-in through an environment variable naming a session id; run it once against a real hub transcript and record the result under Open risks in `internal/orchengine/doc.go`.
- The board copies of the orch stencils are behind the worktree source until a production deploy; the new compact stencil reaches the board only then.
- The `orch-owns-the-loop` board entry's "A watch command" section is covered by this change and needs amending at landing.
