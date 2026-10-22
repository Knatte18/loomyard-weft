---
round: 3
focus:
  - "D4's two session-end orders in `_lyx/discussion/decision-record.md`: `Teardown.EndSession` in `internal/pairteardown/teardown.go` runs the quiet wait, then the refusal probe `Topology.RemoveRefusal`, then the loop end, and its doc says a refused removal leaves the session and its strands live. Settle whether every order D4 states puts the probe ahead of `runproc.End`, and whether the acceptance criteria pin that a refused removal ends no owned process."
  - "D7's site list in `_lyx/discussion/decision-record.md`: `innerRunProducer.ownedElsewhere` in `internal/battenshed/innerrun_owner.go` keeps the parent lock for a Finalize past its parent-side merge (`keepsLockForFinalize`), `pauserecord.SlugEntryExempt` in `internal/pauserecord/hold.go` passes a parent-lock holder, and `PATTERN-owner-machine` bars that Finalize from the owner rule. Settle whether D7's Decision, Bound and acceptance criteria let such a Finalize finish at the batten probe, the wake and `lyx orch start`."
  - "D4's reed down readings in `_lyx/discussion/decision-record.md`: a loom driver runs as a reed strand and holds no driver lock, which only `lyx loom drive` holds, while `pairteardown`'s quiet rule checks the driver strand separately. Settle whether a running, unpaused LLM-driven run can read as not running and be ended by `lyx reed down`."
  - "D4 and D9 in `_lyx/discussion/decision-record.md`: `lyx reed down` in `internal/reedcli/up.go` calls `Engine.Down` unconditionally today. Settle whether the record states, in one place, that a running unpaused pair's session and pane processes survive `reed down` after this task, and whether D9's Bound agrees."
---

Each entry names a site and the question to settle, from the round 2 review's findings; the artifacts are the only evidence.
