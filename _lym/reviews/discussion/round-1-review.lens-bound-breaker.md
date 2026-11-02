```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guarantee and claim about existing code in the record.

**Claims attacked**
- D1, the dead-classification order is lifted from `awaitColdFixer`: held (`internal/websterengine/coldfixer.go:42-46`).
- D1, the strand is registered with runproc, so ending the run stops it: held. A pane pid is already registered this way as `RoleSession` (`internal/loomcli/sharedbootstrap.go:333-335`).
- D1, a restarted caller attaches instead of spawning a second fixer: broken for the card caller (finding 3).
- D2, the wrap covers the closure every evaluation runs: held today. `evaluateGate` picks `Final` over `Gate` and `Forked` on a final evaluation (`wait.go:1650-1656`). None of the three entries sets `Final`, because only PassOnCap or MayHold entries may set it (`gate.go:74`).
- D2, an earlier entry's pass stands: held. Evaluation stops at the first failing entry (`wait.go:1642`). Darn-Write's clean gate runs first and fails on any dirty path (`cleangate.go`). A dirty tree therefore never reaches verify. The verify re-run catches a fixer's leftover file (`verifygate.go:133`). Plan-Write and Discussion-Write have only one entry each (`tapestry-recipe.yaml:122-124,268-270`), so the claim is vacuous there.
- D2, no slot is held while the fixer works: held.
- D2, at most `attempts` plus one fixers per attach: held. Each failed evaluation sends one counted re-prompt.
- D2, a finalize overruns the deadline by at most one fixer timeout plus one inner run: broken (finding 1).
- D2, the `Expired` probe with a record "from before an attach" starts the restart path: broken (finding 2).
- D4, only a call from an agent strand hands off: held. Calls without a strand name, with `--wait`, or inside an enclosing slot never detach (`detachfront.go:232-235`).
- D4, a fixer's own call never joins the run waiting on it: held, because the key hashes the verb, run directory and words (`detachfront.go:304-311`). The runner's environment has the strand name stripped (`detachfront.go:130`), so the caller's role must travel in the request. That is plan detail.
- D4, the "whatever tree identity" join: no bound is stated (finding 4).
- D4, a card gets at most the self-fix cap plus one fixers: rests on a prompt rule (finding 5).
- D4, a cancelled run stops its fixer: held for the lease lapse and terminating signals (`detachrun.go:274-275`). It is broken for a killed runner (finding 3).
- D3, a hub that tunes `card` keeps the template's `fixer`: held (`configengine/doc.go:127`).
- Constraint, an evaluation runs from the first entry: held (`wait.go:1636`).
- Constraint, verify blocks and fails dirty: held.
- Constraint, after an attach only `Expired` probes are asked: held (`wait.go:33-34`).
- Constraint, Plan-Write sets `Forked` after the resolver returns: held (`entries_planwrite.go:76-83`).

**Findings**

1. MEDIUM, design. D2's Bound line on finalize ("can outrun the row's deadline by at most one fixer timeout plus one run of the inner gate") is broken.
   - The deadline exit finalizes and evaluates the gate again (`wait.go:19`, `420`).
   - The wrapped closure runs the inner gate, the fixer, then the inner gate again. A finalize alone therefore overruns by one fixer timeout plus two inner runs.
   - While the closure blocks, the loop reads no deadline (D2's own Bound line). Example: a failing evaluation starts just before the deadline and spends inner, fixer, inner. The deadline exit then finalizes and spawns a second fixer.
   - Real overrun: up to two fixer timeouts plus four inner gate runs.
   - Evidence: `internal/shuttleengine/wait.go:19`, `:420`, `:1650-1659`.

2. MEDIUM, design. D2's `Expired` probe ("answers true when the store holds a fixer record from before an attach") does not limit itself to a live, non-terminal record.
   - Terminal records stay in the store, as webster's `VerifyFixers` do (`verifyfixer.go:80-82`, terminal ones kept with `Status` set). D8 keeps one record set per gate entry on disk.
   - Counterexample: a fixer ends, the re-run still fails, and the writer is re-prompted. While the writer is mid-turn, the driver restarts and attaches. The probe sees the old record and finalizes at once (`wait.go:376-379`, which finalizes even mid-turn).
   - The finalize evaluation then runs the inner gate over the writer's half-done edits, fails, and spawns a new fixer while the writer is still live.
   - This uses up the writer's unspent attempts and breaks D2's rationale that the fixer edits a worktree nobody else is touching.
   - Evidence: `internal/shuttleengine/wait.go:376-379`, `internal/websterengine/verifyfixer.go:80-82`.

3. MEDIUM, design. D1's attach-on-restart and D4's "a detached run cancelled on its lapsed lease stops its fixer" break for the card caller when the detached runner dies without a graceful cancel, for example SIGKILL after runproc's escalation bound, or an OOM kill.
   - The card fixer's record lives in the detached run's directory (D8).
   - A dead runner makes the run unjoinable: `runnerLive` is false (`detachfront.go:316-327`).
   - The fork's next call then starts a fresh run in a new directory with no record. It spawns a second fixer while the first fixer's pane still lives. Two fixers then edit one worktree, and nothing stops the first until the run's processes end.
   - Evidence: `internal/gatecli/detachfront.go:316-327`, `detachrun.go:274-275`.

4. MEDIUM, design. D4 item 3's relaxed join ("a call with the same key joins it whatever the tree identity it sees") has no stated bound on what a mismatched-tree call can join.
   - The key is verb, run directory and words only (`detachfront.go:304-311`). Any strand making the identical call joins regardless of tree, for example the fork, the Master or a recovery strand on the same card.
   - Counterexample: the fork edits a file and calls again during the re-run phase. It joins and receives a green verdict computed over a tree that lacks its edit. It then commits the card on that verdict.
   - Today the tree-identity rule refuses exactly this (`detachfront.go:320`).
   - What stops it is a stencil instruction (`webster-body-implementer.md:110`, "run the same command again in a turn that does nothing else"), and the Decision does not say so.
   - Evidence: `internal/gatecli/detachfront.go:304-311`, `:320`.

5. LOW, design. D4's Bound line ("a card gets at most the fork's self-fix cap plus one fixers") presents a prompt rule as a Go bound.
   - The self-fix cap is stencil text (`webster-template-master.md:317`, `webster-body-implementer.md:117`).
   - Once a run's result is delivered, the run is not joinable (`detachfront.go:319`). A fork that repeats an identical red call on an unchanged tree therefore starts a new run and a new fixer each time.
   - The Decision should state that the bound rests on the fork's prompt cap.
   - Evidence: `contracts/stencils/webster/webster-template-master.md:317`, `internal/gatecli/detachfront.go:319`.

Not verified, so not reported: whether webster's `plan-fixer` strand (`planfixer.go:24`), which D4's never-hands-off list omits, ever calls the card gate.

**Tool calls**
- `wait.go` 1600-1735, grepped: whether an evaluation stops at the first failure. The context lacked the body of the entry loop.
- Webster stencils and roles grep: the self-fix cap's source and the fixer strand roles. These were not in the manifest.
- `entries_gate.go` / `loomshed` grep: to locate the clean gate, which was not in the manifest.
- `cleangate.go` and `wait.go` 1684-1730: what the clean gate checks, and how findings are chosen.
- `wait.go` 1636-1684: the loop head, to check for an early exit and the `Final` selection.
- Master and fork stencil grep: whether forks share one worktree concurrently. Nothing was found.
