---
round: 3
focus:
  - "Check the decision record's D6 against every halt that reports a failed verify command run through `lyx gate test`: webster's verify gate (`internal/websterengine/verifygate.go`), the loom verify gate (`internal/loomshed/verifygate.go`, refusal-spec rows `Webster-Burler-Fix verify gate exhausted` and `Darn-Write verify gate exhausted`) and the landing verify gate (`internal/landingshed/verifygate.go`). Settle whether each halt reason, when the log carries D7's watchdog line, names raising `busy_quit_sec` as its way forward, and whether Scope and Acceptance criteria cover each."
  - "Check D7's skip of the watchdog under an enclosing `lyx gate test`. Settle what bounds the skip: whether a process that sets the enclosing-gate-test variable by hand, or a detached descendant such as a tmux server's pane that inherits it, can run `lyx gate test` with no watchdog and no outer watchdog watching it."
  - "Check the Acceptance criteria that drive an inner `lyx gate test` from a test that itself runs under `lyx gate test`. Settle whether the inner run's own 1-second window is what the criterion asserts, or whether the outer run's watchdog at the hub's `busy_quit_sec` could satisfy it."
  - "Check D7's 90 percent of one core threshold against the criteria that test it. Settle whether any criterion's pass depends on machine load, and whether a single spinning goroutine is pinned by an injected sampler rather than by a live fixture."
---

Focus targets the open design questions on watchdog way forward, the skip's bound, and criterion determinism.
