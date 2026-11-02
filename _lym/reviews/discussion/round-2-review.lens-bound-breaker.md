```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I tried to break every bound, guard, ordering promise and claim about existing code in the decision record.

**Claims attacked**
- D2 rationale, "the writer sits idle at its turn boundary, so the fixer edits a worktree nobody else is touching": **broken** (F1).
- D2 bound, a row run outruns its deadline by at most two fixer timeouts plus four inner gate runs: the arithmetic holds, but the path it counts is the one F1 breaks.
- D2 bound, at most `attempts`+1 fixers per attach: held. When the budget is spent, finalize reads the memoised verdict (`wait.go:1609`), so it does not re-run the gate.
- D2, an earlier entry's pass stands because Darn-Write's verify gate fails on a dirty tree: held. `cleangate.go:18` and `verifygate.go:133` read the same dirty-path check.
- D2, the `Expired` probe finalizes an attached run that holds a live fixer record: held as far as the shuttle loop goes (`wait.go:372-380`). The store side is broken (F3).
- D1, ending the run's processes by cancel, goto or reset stops the fixer: **broken for goto** (F2).
- D8 and D1, a caller that restarts attaches to the live fixer: **not guaranteed**, because D8 keys records to the row visit without defining it (F3).
- Constraint, no shipped gate closure returns `Pending`: held. The only non-test `Pending: true` is in creel (`creelengine/server.go:392`), which is not a gate.
- Constraint, an evaluation runs every entry from the first: held (`wait.go:1636`).
- Constraint, Plan-Write sets `Forked` after the shared resolver: held (`entries_planwrite.go:81-84`).
- D4, only a capped call hands off: held (`detachfront.go:232-258`). Capping also needs a keep-warm cap above zero, which only narrows the set further.
- D4, the runner's environment carries no strand name: held (`detachfront.go:130`).
- D4, a fixer strand keys its own run by strand name, so it never joins the run waiting on it: held.
- D4, the fork stays idle while the card fixer works in its worktree: held. Forks run one at a time in one worktree (`websterengine/doc.go:4`).
- D4 rationale, the gate verb learns nothing of webster: **broken** by the record's own guard (F4).
- D4 and Scope Out, the darn writer's own `lym gate test` runs are out of scope: **breakable by config** (F5).
- D4, a dead runner's fixer is attached to or stopped by the next run of the key: held, as long as the caller runs the same command again, which the fork stencil tells it to do.
- Constraint, 30 minutes stays above twice the longest call: arithmetically held (2 × 594 s ≈ 19.8 min).

**Findings**

F1. BLOCKING, design. D2, step 2 and the Rationale (`decision-record.md:57`, `:77`, `:80`), with acceptance criterion `:215`.
A deadline finalize spawns a fixer while the writer is mid-turn, so two agents edit the same worktree.
- The counterexample:
  - An evaluation starts before the deadline, fails over the fixer and re-prompts the writer with budget left (`wait.go:507-509`).
  - The writer starts editing.
  - On the next tick, `wait.go:416-420` finalizes on the deadline, whatever state the writer is in.
  - `finalize` runs `evaluateGate(true)` (`wait.go:1781-1784`) before the strand is removed (`wait.go:1833`).
  - The wrapped closure fails over the writer's half-done edits and spawns a fixer next to the live writer.
  - The re-run then judges a tree both agents are changing, and on Darn-Write the writer may commit at the same time.
- D2 counts this very fixer in its deadline bound.

F2. MEDIUM, design. D1, second bullet (`decision-record.md:37`), with D8 (`:175`).
A goto does not end the run's processes, so a live fixer outlives it.
- Evidence:
  - `internal/shedverbs/goto.go` contains no runproc reference and no stop call.
  - The runproc end call sites I found are tender (`tendercli/deps.go:113`, `:321`), pair teardown (`pairteardown/teardown.go:603`), fabric (`fabriccli/anchordeps.go:243`) and dobby `EndRole`.
- The counterexample:
  - The driver crashes while the closure blocks on a fixer.
  - The operator runs a goto to another row.
  - D8 starts an empty record set for the new visit, so nothing ever reads the old record.
  - The fixer keeps editing the worktree under the new row's agent, and nothing stops it.

F3. MEDIUM, design. D8, first bullet (`decision-record.md:175`), against D2's restart path (`:73-74`) and acceptance criterion `:217`.
D8 never says whether a crash re-step of the same row counts as the same visit or as a row re-entry.
- The resume path re-steps an interrupted step (`tendercli/deps.go:311`, `sheddriver.RestepRecorded`).
- If that re-step starts an empty set:
  - The `Expired` probe and the wait-only entries never see the live fixer.
  - Darn-Write's `clean` entry fails on the fixer's uncommitted edits, which is round-1 F1 regressing.
  - A second fixer can spawn next to the first.

F4. MEDIUM, consistency. D4's Bound (`decision-record.md:112`) against D4's Rationale (`:121`), with D1 (`:36`).
The hand-off guard tests webster's `verify-fixer` role, so the gate verb does learn of webster.
- `verify-fixer` is a websterengine constant (`verifyfixer.go:26`).
- PATTERN-agent-name says a spawned role is its module's constant.
- To test it, gatecli must either import websterengine or repeat the literal.
- The record does not say where the set of fixer roles lives.

F5. LOW, scope. D3 (`decision-record.md:88`) and D4 (`:99`), against Scope Out (`:26`).
`fixer` is accepted on any gate.yaml key, with no stated bound.
- If a hub sets `gates.darn-write.fixer` (or the field on `drift-verify` or `code-burler-fix`), every capped call those agents make during their work spawns a fixer.
- That is exactly what Scope Out excludes for the darn writer.
- Neither D3 nor D4 refuses the field on other keys, or names that consequence as accepted.

**Tool calls**
1. Grepped the webster fork stencil and `websterengine/doc.go` for concurrency: my context did not show whether forks share a worktree at the same time.
2. Grepped for non-test `Pending: true` and for runproc end call sites: needed to check the no-`Pending` constraint and who ends a run's processes.
3. Read `tendercli/deps.go` around both `runproc.End` sites: needed to see which verbs end a run's processes and how resume re-steps.
4. Grepped `shedverbs/goto.go` and `shedengine/goto.go` for runproc or stop calls: needed to check D1's goto claim.
5. Grepped loomcli and battencli goto code for process ending: needed to rule out goto ending processes elsewhere.
