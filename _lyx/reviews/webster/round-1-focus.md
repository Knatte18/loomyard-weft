---
round: 1
focus:
  - "internal/hibernateengine watch sequence: does the daemon pause, confirm the pause, hibernate, detect the wake and lift only its own pause entry, in that order, with the injected wall and awake clocks and no real systemctl or production wait in tests?"
  - "internal/tenderengine Lift and the pause callback: does Lift remove exactly one entry by identity, and does the callback name the entry the pause added, without changing existing pause behavior?"
  - "Shared decision: does the engine reach tender only through the Pause, TenderStatus and Lift seams, with the request file and log under the hub's _board/.lyx/hibernate directory?"
  - "internal/claudeengine sleep-deny: are all machine sleep commands denied for every spawned session, and does a test cover each denied command?"
  - "Test economy across the new hibernate, tender and shortcut tests: any duplicate coverage, per-helper test already covered by a public-surface test, or added behavior with no test?"
---

Seed focus drawn from the plan's card index and shared decisions.
