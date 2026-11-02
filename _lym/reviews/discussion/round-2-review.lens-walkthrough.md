```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's stated purpose.

**Walkthrough list**

- **D1** (core spawn, wait, attach, stop). Case: the Darn-Write fixer ends with its reply file written, so the wait reads done, stops the strand and removes the run dir. The `awaitColdFixer` order carries over as written. **Purpose met.**
- **D2, normal boundary path.** Case: Darn-Write at a turn end. The `clean` entry passes, since there is no live record. `verify` fails, so the report is written, the fixer is spawned and the closure blocks. The fixer commits `fix:` and leaves the tree clean, the verify re-run passes, then `description` runs. **Purpose met.**
- **D2, deadline-finalize path.** Case: Plan-Write. The writer was re-prompted and is mid-turn when `run.deadline` passes (wait.go:416-420). `finalize` calls `evaluateGate(true)` (wait.go:1784) over the writer's half-written plan, the plan gate fails, and the wrap spawns a fixer into `_lym/plan/` while the writer is still editing there. **Purpose missed** (F1).
- **D2, restart path.** Case: the driver dies while the `verify` closure is blocked. On attach, the `Expired` probe fires off the boundary (wait.go:372-379). The `clean` wait-only wrap waits on the live record and then judges the finished tree. **Purpose met**, but it depends on D8's record keying (F2).
- **D3.** Case: `fixer: darn.gate_fixer` resolved through `Env.Setting` at build, the way `parts_max` is resolved (entries_gate.go:201). **Purpose met.**
- **D4.** Case: the fork's capped `lym gate test --gate card --card 3`.
  - The front parses the role and the runner runs the gate and spawns the fixer.
  - The fork's repeat calls join through the fixer phase and the re-run, and the call after the re-run's exit replays its result.
  - The fixer's own call is keyed by its strand name and never hands off.
  - **Purpose met**, apart from the fixer's parent directive input (F4).
- **D5.** Case: a forked Plan-Write evaluation whose only blocking finding is `lens-review-missing`. Go can detect it from the finding's `Check` code (gates.go:319, 328), so no fixer is spawned. **Purpose met.**
- **D6.** Case: `fix-verify` hands its `VerifyFixers` state to the core as the store, and the record is attached and marked terminal as today (verifyfixer.go:81, 198-207). **Purpose met.**
- **D7.** Case: Discussion-Write with the fixer on. The writer writes the files, runs its lens review, and ends its turn. The gate fails and the fixer edits the files the lens review already read. **Purpose met**, but the staleness goes unstated for Discussion (F3).
- **D8.** Case: a goto re-enters Darn-Write in the same run after a fixer was live. The record set should start empty, but no identity of a row visit exists to key it by. **Purpose not shown to be met** (F2).

**Findings**

F1: BLOCKING, design. Location: D2 (decision-record.md:56-57, 74, 77, 80) and acceptance criterion line 215.
- **Summary:** a finalize evaluation off the writer's idle boundary spawns a fixer into a worktree a live writer is still editing.
- **Walk:** at the deadline, the loop calls `finalize(classifyDeadlineExpiry(...))` whatever the writer is doing (wait.go:416-420). `finalize` runs `evaluateGate(true)` before it removes the strand (wait.go:1781-1793, then 1833). The wrapped `Gate`/`Forked` closure is the one evaluated, because a must-pass entry cannot set `Final` (gate.go:56-58; wait.go:1625-1626). It takes no argument (gate.go:44), so it cannot tell a final evaluation from a boundary one.
- **Result:** on Plan-Write and Discussion-Write, whose rows have a single entry and no `clean` gate (tapestry-recipe.yaml:122-124, 268-270), the fixer and the re-prompted writer edit the same files at once. On Darn-Write the same happens whenever the writer's tree is clean at that moment. The re-run then judges a moving tree.
- **Contradiction:** D2's rationale says "the writer sits idle at its turn boundary". D2's bound and the acceptance criterion at line 215 still count this deadline-finalize fixer.
- **Evidence:** wait.go:416-420, 1764-1793; gate.go:44, 56-58.

F2: MEDIUM, design. Location: D8 (decision-record.md:174-175), with D2's `Expired` probe (line 73).
- **Summary:** "the set is keyed to the row visit" rests on a row-visit identity the record never names, and none exists.
- **Walk:** the only scratch dir the row Env carries is per run: `shedrun.ScratchDir(location, runID)` (shedrun/paths.go:155-156, wiring.go:627, recipe.go:91-93), and a grep of `shedrun`/`shedengine` finds no visit identity.
- **Result:** on a goto that re-enters the row, a record left without a terminal status reads as live, because the goto's process end stops the strand without marking the record (D1). The new visit's wait-only wraps then wait on it, and a probe keyed on "saved before the run's last attach" could finalize the new visit.
- **Fix:** a deferred research question. Name the identity, such as a counter of row entries, with evidence for where it lives.
- **Evidence:** internal/shedrun/paths.go:155-156; internal/shedrecipe/recipe.go:91-93.

F3: LOW, consistency. Location: D7 (decision-record.md:163-165).
- **Summary:** D7 states and accepts the lens-review staleness only for Plan-Write. The Discussion writer also "runs its lens review right after writing", and its fixer then edits the three discussion files that review read, but the record says nothing about that staleness or what judges it.
- **Evidence:** decision-record.md:163 and 165; D5 line 135.

F4: LOW, consistency. Location: D4 step 1 (decision-record.md:101) with D1 (line 38).
- **Summary:** nothing hands the card fixer's parent name to the detached runner that spawns it.
- **Walk:** D1 renders the parent directive "as the webster cold fixer renders them". That renderer takes a parent name, and an empty name renders the no-parent variant (render.go:372-373, 387). The runner's environment drops the strand name (detachfront.go:130), and D4 has the front write only the caller's role into the request (line 111). So the card fixer always renders the no-parent variant, and the record does not say that this is the intent.
- **Evidence:** internal/gatecli/detachfront.go:130; internal/websterengine/render.go:372-387.

**Tool calls**

1. Grepped `shedrun`/`shedengine` for a row-visit identity: my context had no definition of the "row visit" D8 keys the record to.
2. Grepped `shuttleengine` `spec.go`/`reed.go`/`run.go` for parent naming: my context lacked how a spawned strand learns its parent.
3. Searched for the parent directive renderer and a row-entry identity: the first two calls left both open (no output).
4. Located the parent directive source and the `shedrun.ScratchDir` definition: needed to confirm the scratch dir is per run.
5. Grepped `parentName` in `websterengine/render.go`: needed to confirm what the cold prompt's parent directive takes as input.
