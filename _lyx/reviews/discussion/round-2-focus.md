---
round: 2
focus:
  - "Decision D3 (pending compact or clear request removal): settle whether the stated test removes exactly the requests the finished cycle covered, given that `internal/orchengine/watcher.go` `enter` overwrites `PhaseEnteredAt` on every phase entry and a compact cycle passes handoff-requested, compacting and resuming."
  - "Decision D7 (batten-watched run marker): settle whether the place where the fact is recorded is compatible with `internal/shedrun/seed.go` `WriteSeed`, which is write-once and compared for equality against the seed the child's own bootstrap writes (see the comment on `childSeedParams` in `internal/battencli/wire.go`)."
  - "Decision D7 and D5 (removal of the driver's run-stopped message): settle what bounds the case where no batten notice replaces it, covering a batten that dies after the driver prompt was rendered, a notice that fails to send, and `QueueNotice` returning not-queued when no orch strand is recorded (see `internal/battenshed/notice.go` `noticeStep` and `internal/battencli/wire.go` `Notify`)."
  - "Decision D8 (batten window in the reed session): settle how reed keeps enumerating, laying out and reaping only its strand panes once a second window exists, given that `internal/reedengine/overlay.go` `listPanes` targets the session's current window and `internal/reedengine/doc.go` states every pane in a reed session is Selvage or a bound strand pane."
  - "Decision D8 window lifecycle: settle what happens to the window when batten exits given that reed sets `remain-on-exit on` for every session (`internal/reedengine/lifecycle.go`), what stops a second batten for one slug, and how the window is addressed."
  - "Decision D5 (state-changed notice contents): settle whether the stop report path and the since value are tied to the stop that changed state, given that the drive-reports directory is durable and keeps earlier stops' reports and a park marker may predate the stop."
  - "Decisions D4 and D5 ordering: settle whether a pending notice is guaranteed to be sent before any return of the wait call, including the done arm returning `Done` when the driver strand ends, after which batten exits."
  - "Decisions D4 and D6 bounds: settle which stalls bound an unlimited wait on a running child once the 12-hour window is gone, and what D6's activity test does for unreadable transcripts, several live runs, dead pids, a busy run with no step progress and a hung live wait."
  - "Decision D6 api-error notice: settle whether provider-derived text queued into the orch session is bounded in length and shape, given that `QueueNotice` refuses a line with a newline or a leading `/`."
  - "Decision D4 seams and rates: settle whether batten's own pause flag path is a named injected seam, how a failed stat of the child's status file differs from a decode failure, what a running child with a dead driver and an unchanged status file waits on, and how often notice probes run against the 2 s check."
  - "Decision D7 rationale versus D5: settle whether an awaiting child produces one message or two, given D7 keeps the driver's parent_notice relay for an awaiting envelope and D5 lists awaiting among the states batten notifies."
---

Entries name the question only; each is to be judged against the discussion pair as it now stands.
