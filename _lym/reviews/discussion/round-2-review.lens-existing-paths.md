```yaml
lens: existing-paths
outcome: reported
```

**Paths traced, per decision**
- D3 — Finalize `rerouteReviewed` / `codeBudgetZero`: read only loom.yaml's code cap (`internal/loomcli/landingdeps.go:390-409`); D3 states it changes them to follow the entry gate too — survives (as stated).
- D4 — creel notice after `decision add`: excludes the caller's own run, so the sampler agent gets no echo of its own entry — survives.
- D4 — record commit's detached both-sides push: `Fabric.Commit` fires a both-sides push in a task pair — meets D5's steer text, see F1 — breaks D5.
- D5 — publish guard's steer: one constant `steerPublishDeny` on one `SwitchDenyPublish`, turned on by a `DenyPublish bool` on the spec; no per-spec steer path — breaks.
- D5/D6 — session step tie: `run.go:693` copies the step into the run record, and `StopStepRuns` stops tied runs on a watchdog kill; D7 exempts the row from that kill — survives.
- D6 — entry path vs a live recorded session: interrupt leaves the session live (PR-Review shape), and `lym reed resume` replays not-live strands while the run is halted — breaks.
- D6 — stamp vs Relaunch: `goto.go:249` writes a Goto onto the target; a relaunch onto the halted row is "from" the same row, stamp unchanged — consistent with stated rule — survives.
- D6 — stamp vs reroute/on_stuck entries: no sampler row routes to Sampler-Session (darn-recipe.yaml edges copied) — survives.
- D6 — record-check seam: `darnDeps.CheckRecord` reads history via `runRecordOf(l, shedrun.RecipeDarn)` (`darnwiring.go:163-166`); D6 rewires per row — survives (as stated).
- D7 — dobby: `childAtWaitingRow` is the one quiet-notice reader — survives.
- D7 — watchdog: `agentActivityFor` has no row reading today; D7 adds one — survives (as stated).

**F1 — MEDIUM — design — D5 (publish guard steer)**
D5's steer says "lym pushes the branch … only after `sampler-done` ends the row". In a task pair every run-record commit fires a detached both-sides push that carries the code branch, with the agent's commits, to origin; D4's own `decision add` run by the agent is such a commit, so the code branch reaches origin mid-session. Evidence: `internal/fabricengine/doc.go:222-228`; the publish row's own comment `internal/shellguard/rows.go:247-248` ("the detached both-sides push a run-record commit fires, reaches origin from the session too").

**F2 — MEDIUM — design — D5 ("steer text of its own")**
The publish row has one fixed steer, `steerPublishDeny`, and a spec can only turn it on (`DenyPublish bool`); `claudeengine` maps `SwitchDenyPublish` to that single row. A sampler-specific steer needs a new row/switch or a parameterised steer in the guard table, which `PATTERN-shell-guard-table` requires every engine to render — D5 names no such change, and its bound names a steer text the code does not provide. Evidence: `internal/shellguard/rows.go:19-22,99-103,249-253`; `internal/shuttleengine/spec.go:55-57`; `internal/shuttleengine/claudeengine/settings.go:280`.

**F3 — MEDIUM — design — D6 (entry bullet and Bound)**
On entry a current-stamp flag with a clean checkout and an intact record ends the row Done "without starting or resuming a session" — nothing stops a recorded strand that is already live. Two existing paths leave it live while the row is not stepping: PR-Review's `endSession` keeps the session on a cancelled context (`prsession.go` `endSession`), and `lym reed resume` replays not-live strands (`internal/reedcli/resume.go:1-3`), which D6's own Bound says may reopen the session while halted. Counterexample: halted → `lym reed resume` reopens the agent → someone runs `sampler-done` (accepted while halted) → `lym loom resume` reads a clean checkout, ends Done, agent still live and free to change the checkout after the check, into Describe and Finalize. The wake path stops the session before its check; the entry path does not.

Tool calls:
1. grep `shuttleengine/run.go`, `reedcli`, `fabricengine` — whether an unawaited session carries a step tie, what `reed resume` does, whether a record commit pushes; context lacked all three.
2. sed `fabricengine/doc.go:220-250`, `run.go` step lines, `reedcli/resume.go` doc — confirm both-sides push and replay semantics.
3. grep callers of `StopStepRuns` and automated `reed resume` — which paths stop or revive the session.
4. grep `rerouteReviewed`/`codeBudgetZero` and darn wiring — their location was not in context.
5. sed `landingdeps.go:380-420`, `darnwiring.go:140-170` — read the perch-off and record-check seams D3/D6 change.
6. grep `review_dir` in landingshed/loomcli — how Describe fills it when the perch is skipped (no hit; no finding).
