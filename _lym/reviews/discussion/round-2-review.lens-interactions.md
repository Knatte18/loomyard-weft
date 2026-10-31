```yaml
lens: interactions
outcome: reported
```

**Pairs checked**
- D1 × D3 (perch rows' `entry_gate: sampler.code_review` vs Finalize `on_reroute: Code-Bouncer`): held — D3 makes `rerouteReviewed`/`codeBudgetZero` read a gated-off perch as off.
- D1 × D3 (Describe no `entry_gate` vs Code-Review budget 0): held.
- D2 × D3 (profile `sampler` blocks vs `run_settings` vs `sampler.yaml` default): held.
- D1 × D6 (entry stamp vs the recipe's routes into Sampler-Session): held — only Record-Write's `on_done` and a goto reach the row; no `on_stuck`/`on_reroute` names it (`contracts/recipes/darn-recipe.yaml`); a relaunch onto the same row writes a goto entry (`internal/shedengine/goto.go:249`), which "from another row" correctly does not count.
- D4 × D5 (caller rule's admitted strand vs the row re-recording on resume/start): held on the row's own paths.
- D4 × D6 (admitted strand vs the `lym reed resume` reopen D6's bound allows while halted): **broken** — see F2.
- D5 × D6 (session stop before the checkout check, wake path): held.
- D5 × D6 (entry path with a current-stamp flag while a recorded strand is live): **broken** — see F1.
- D5 × D6 (dirty-on-entry: ensure session then mail): held — mail follows ensure.
- D6 × D6 (flag written while halted, goto away and return): held — goto back raises the stamp.
- D6 × D6 (pause covered → flag → first uncovered wake): held — resume then read flag then stop; wasteful but correct.
- D6 × D7 (watchdog exemption at the row vs the row's own dead-strand halt): held — the row's wake still halts on a dead strand.
- D6 × D4 (record check after the agent's `decision add` commits): held — session stopped first.
- D8 × D4 (stencil "settles that changes the task's scope" vs D4 "what the operator and the agent settle"): held — D4 bound reads the stencil as the limit.
- D8 × D9 × D6 (kill the `tmux -L` server before `sampler-done`): held.
- D5 steer × D4 (`decision add` commit pushing branch mid-session): held — `commitRunRecord` → `commitDiscussion` → `fabricengine.CommitAnchoredPaths`, a no-push commit (`internal/fabricengine/commitweftpaths.go:92-96`).
- Acceptance D6 "flag written while halted … goto away and returned": held against D6's stamp rule.
- Acceptance D6 "on a clean checkout it ends the row Done and the session is stopped": **broken** by F1's entry path (ends Done with no stop).
- Acceptance D4 "also after the session was resumed under a new strand": held only for row-driven resumes (F2).
- Constraints `PATTERN-loom-contract`, `PATTERN-shed-recipe-registry` vs D1/D3/D4/D6: held.

**F1 — MEDIUM — design**
Location: `decision-record.md` D6, the "On entry, before it ensures a session" bullet, against the wake bullet and D6's Bound.
Summary: the entry path ends the row Done on a current-stamp flag with a clean checkout "without starting or resuming a session", but never stops a recorded strand that is still live. D6's own bound says the stop chain or `lym reed resume` may reopen the session while halted at the row, and the verb is accepted while halted (from the agent or the operator's shell). The next `lym loom resume` then ends the row Done on the entry check with the reopened agent still running, free to change the checkout after the check and to keep working through Describe and Finalize — breaking D6's "Describe and Finalize always start from a committed tree" and contradicting the D6 acceptance bullet "the session is stopped". The wake path stops the session first; the entry path needs the same stop before its checkout check. (PR-Review's `endSession` also leaves the session running on a cancelled context, `internal/landingshed/prsession.go:219-228`, a second way a live strand reaches entry.)
Evidence: `decision-record.md` D6 entry bullet and bound sentence; `internal/landingshed/prsession.go` `endSession`.

**F2 — LOW — consistency**
Location: `decision-record.md` D4 ("so the admitted strand is always the live one") vs D6's Bound.
Summary: D4 admits only the strand the row recorded, and the row re-records only on its own resume/start; a session reopened while halted by `lym reed resume` or the stop chain (D6 bound) is not re-recorded by the row, so that live agent's `decision add --by operator` is refused while the stale recorded strand is "admitted". D4's "always the live one" does not hold on that path, and the record never says which outcome is intended.
Evidence: `decision-record.md` D4 second paragraph and D6 bound; row re-recording only in `ensureSession` shape (`internal/landingshed/prsession.go:59-99`).

Tool calls:
- grep `fabricengine.CommitAnchoredPaths` push behaviour — context lacked whether a record commit pushes the task branch.
- read `internal/fabricengine/commitweftpaths.go` 85-140 — confirm the commit shape is no-push.
- grep commit seam in `internal/loomcli/decision.go` — find which commit `decision add` uses.
- read `commitRunRecord` in `internal/loomcli/projectrecord.go` — follow that seam one hop.
- read `commitDiscussion` in `internal/loomcli/wiring.go` — confirm the task-record commit is the no-push one.
