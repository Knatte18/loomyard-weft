---
round: 1
focus:
  - "Cards 5 to 8 and the rate-of-reads decision: check in the Run-Shed wait that file stats and small reads run per `poll_interval_s` check, and that process or multiplexer reads (reed status, agent activity, wait markers, forced decode, resume retry, watched-marker refresh, queue retry) run at most once per `notice_probe_s` off one shared probe-due clock."
  - "The battenshed-stays-neutral decision: check that `battenshed` imports only `shedengine` and `logger`, and that every new reading arrives through an `InnerRunDeps` or `SeedChildDeps` seam wired in `battencli/wire.go` with `battenshed`'s own neutral types."
  - "Card 7 and the episode-state decision: check that `since` and the watched-marker answer live in the producer value across calls of one batten process, that only a batten start reads `since` from the status file, and that each stop's notice is sent once."
  - "Card 2 and card 1: check that a same-mode request queued during an orch cycle is removed exactly when the cycle's action takes effect, and that the compacting phase completes on a boundary `CompactedSince` finds."
  - "Cards 9 to 11: check the reed strand window is addressed by id through one seam, the detached named-window op runs `lyx`, and `lyx batten run <slug> --window` plus the orch stencil's start step match the card; run the per-card mechanical checks the plan spec lists for each group, against every target."
  - "Test economy: for each new test, check it against the existing tests covering the same behavior, and check each added behavior (api-error notice, watched marker, wait events) for coverage."
  - "Comment conventions: check new doc comments against the repository's own constraints document and `doc.go` module docs, since the task changes observable CLI behavior."
---

Focus lists where to look; it sets no verdict.
