```yaml
lens: existing-paths
outcome: reported
```

I ran the existing-paths lens: for each piece of state the record introduces, I traced the code that already writes, resets or reads it.

**Paths traced, per decision:**

- D4, capture process and `runproc`:
  - `pairteardown.endOwnedProcesses` runs `EndSparing` and ends the capture: **survives** (the Constraints bullet states this).
  - `tender reap`: an existing pair is never an orphan, so the capture is never reaped: **survives**.
  - `EndOwnedElsewhere` on adopt: **breaks** (F1).
  - `runproc.List` prunes the record of a dead capture: **survives**.
- D4, `.lyx/drift/` placement: `.lyx` is a junction into the task's weft, already excluded by `WireJunctionsWith` at clone. It **survives** as dirt-free, but the record restates this as new work (F5).
- D4, the timeout clock across processes: a machine sleep or hibernate while `Drift-Await` waits **breaks** (F4).
- D5, `awaited_at` on Webster's reroute through the Plan perch into `Drift-Await`: **survives**.
- D5, the way forward `lyx loom goto Drift-Baseline` meeting Plan-Write's fresh-spawn preparation: **breaks** (F2).
- D5, `awaited_at` on adopt to another machine: **breaks** (F1).
- D6 and D7, the accepted list and run counter on re-entry through loom Finalize's `on_reroute` (Parent-Align, Code-Bouncer, `Drift-Verify`) and on `lyx loom resume` after a halt: **breaks** (F3).
- D6, code changes after `Drift-Verify` (loom Finalize's merge fixer, darn Finalize's reroute to Code-Bouncer): **breaks** the "no code change goes uncompared" claim, and the record is silent (F6).
- D2, entry gate `drift.enabled`: `Env.SettingKnown` is filled in both `loomcli/wiring.go:765` and `battencli/wire.go:1424`: **survives**.

**Findings:**

**F1. MEDIUM, design. D5 and D6, adopt to another machine.**
- `.lyx` is a symlink into the task's weft `.lyx`, which is never tracked. The baseline directory, `capture.json` and the run counter all live there, so an adopted worktree has none of them.
- When another machine owns the run, `EndOwnedElsewhere` ends any running capture.
- D5 says `awaited_at` lets "an adopt on another machine pass without a baseline check". But `Drift-Verify` then finds no passed capture record and halts Stuck, naming `goto Describe`. Regression is lost, and no recapture is possible once code has changed.
- The record never says what an adopt does to regression.
- Evidence: `internal/tendercli/deps.go:263-268`, `readlink .lyx` → `…-weft/.lyx`, `decision-record.md:120` and `:127`.

**F2. MEDIUM, design. D5's way forward `lyx loom goto Drift-Baseline`.**
- In loom, `Drift-Baseline`'s `on_done` is `Plan-Write`.
- When Plan-Write starts a fresh lead, it first runs `NewPlanDirRotator`, which archives the approved plan. The whole Plan perch then runs again.
- So the halt's recapture route silently costs a full re-plan and review, and the record does not say so.
- Evidence: `internal/loomshed/planwrite.go:96-100`, `decision-record.md:53` and `:118`.

**F3. MEDIUM, consistency. D6 and D7 state on re-entry into `Drift-Verify`.**
- Loom Finalize's `on_reroute: Parent-Align` leads back through Code-Bouncer into `Drift-Verify`. A halt's own way forward ("`lyx drift accept` and resume") re-enters it too.
- The accepted list persists in `scope.json`. The acceptance criterion "Under scope `full`, any acceptance ends `Drift-Verify` Stuck" then halts every later entry, even on a passing compare. That includes the operator's own accept-and-resume, so that way forward loops.
- D6 instead says "a session that accepted", and the scope record does not record who accepted.
- Separately, the counter is "per agent session", but no reset is stated for re-entry.
- Evidence: `contracts/recipes/loom-recipe.yaml` Finalize `on_reroute: Parent-Align`, `decision-record.md:135`, `:142-143` and `:194`.

**F4. MEDIUM, design. D4 and D5 timeout measured across processes.**
- The capture times itself on its own monotonic clock.
- `Drift-Await` runs in a later process, so it can only judge "`capture.timeout_min` plus one minute from its command start" by the start time written in `capture.json`, which is a wall time.
- After a sleep or hibernate, wall time advances while the capture's monotonic timer does not. `Drift-Await` then halts Stuck on a capture that is still within its own timeout.
- Evidence: `decision-record.md:99`, `:101` and `:114`.

**F5. NIT, consistency. D4.**
- "The task worktree's `.lyx/` is kept out of git status through `.git/info/exclude`" is already done by `WireJunctionsWith` when the clone is made. The sentence reads as new work for the plan.
- Evidence: `internal/fabriccli/clone.go:113-116`.

**F6. LOW, design. D2 and D6.**
- D6 claims "no code change goes uncompared", but later rows still change code:
  - loom Finalize's merge and merge fixer run after `Drift-Verify`;
  - darn Finalize's `on_reroute: Code-Bouncer` goes back past darn's `Drift-Verify`.
- D2 names only darn's perch fixes as uncompared.
- Evidence: `contracts/recipes/darn-recipe.yaml:34-35`, the Finalize rows in `loom-recipe.yaml`, `decision-record.md:54` and `:140`.

**Tool calls:**
1. Searched for `info/exclude` and `RemoveAll` on ephemeral dirs, to see whether anything already excludes or wipes `.lyx`. The manifest had neither.
2. Read `clone.go:100-125` and resolved the `.lyx` symlink, to learn where `.lyx/drift/` actually lives.
3. Read tender's `EndOwner` callers and `reap.go`, to see whether a reap ends a detached capture between passes.
4. Read tender's orphan rule in `doc.go` and `processes.go`, to confirm an existing pair is never reaped.
5. Read `processes.go` `orphanOwner` (same call as 4), for the role check.
6. Searched for how `_lyx` run records reach the parent at landing. It found nothing, so that path is unverified and not reported.
