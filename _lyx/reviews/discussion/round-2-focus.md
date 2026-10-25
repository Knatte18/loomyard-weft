---
round: 2
exclude_lenses: []
focus:
  - "Wake and ownership: in decision-record.md D3 and D4, a dobby that lives through a machine sleep runs short pass processes; the existing owner check in internal/battenshed/innerrun_owner.go becomes due only on a sleep the same process watched, a pause lift or a retry while owned elsewhere. Settle whether the first pass after a wake runs the origin reconcile, as PATTERN-owner-machine requires, and whether every claim in D3 and D4 about who covers a takeover by another machine is true under the chosen mechanism."
  - "Pass state across processes: in decision-record.md D4 and Open risks, settle whether the state that today lives in InnerRun producer fields and in the per-Call childWait (lock watch, idle watch, retry times, notice latches) is inventoried with its source and persisted across one-check passes, so the idle bound and once-per-episode claims hold, and whether the Constraint about surviving restarts is accurate."
  - "Removal and cancel: in decision-record.md D5, settle whether any path other than an explicit release leaves a task statusless, including internal/pairteardown/claim.go settleClaim and lyx fabric remove, and whether lyx reed down (internal/pairteardown/skip.go EndBeforeSessionDown) ending a beam's dobby is bounded in D1 and D2."
  - "Escape hatches and pass bounds: in decision-record.md D4, settle what the hidden --continue-pass flag can skip and what bounds who may pass it, and whether a pass can stay in flight through in-process sleeps (holdSpawnWhileCovered, the driver's PauseHold, Land polls, prime-lock wait) so that D12's lift pass is skipped."
---

Findings below MEDIUM from round 1 are not promoted into focus; the fixer still addresses them.
No lens is excluded: round 1 ran every lens, but the gating areas were fixed with new text no fresh reviewer has seen.
