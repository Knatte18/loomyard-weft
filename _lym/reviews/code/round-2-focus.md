---
round: 2
focus:
  - "internal/hibernatecli/deps.go hubDeps and internal/orchcli/notify.go NotifyPrime: the wake notice must reach the orch by its real creel name. Settle how the receiver is resolved when reed no longer holds the recorded orch GUID, whether the Notify seam resolves the current name at each retry attempt in sendNotice (internal/hibernateengine/watch.go), whether a pinning test exists in internal/hibernatecli, and whether a notice dropped for want of a recorded orch strand is logged at Warn."
  - "internal/hibernateengine/watch.go notice-file routing (a multi-line text under noticeLineBound, a notice-file write failure falling back to the log) and the fail notice built from a long reason: settle which of these branches have a TestWatch row in internal/hibernateengine/watch_test.go that exercises them."
  - "internal/hibernateengine/watch.go Watch and wake.go reconcileAtWake: settle what happens on a wake whose request has no Owned but whose kept entry is set — whether the reconcile runs, whether the entry is lifted, and whether the notice text is consistent — and whether a row covers it."
  - "internal/hibernateengine/watch.go finish(PhaseDone): settle the behavior when the done-phase write fails after a successful lift — entry lifted, wake notice sent, error returned — and whether a row covers it."
---

Round 1 is not converged; the focus names where the gating design findings sat and which questions the fixes must settle.
