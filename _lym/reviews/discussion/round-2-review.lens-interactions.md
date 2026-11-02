```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens, which checks pairs of decisions against each other, with acceptance criteria and constraints checked against the decisions they touch.

**Pairs checked:**
- D1×D2 (when a fixer record is marked terminal, and the restart attach): held.
- D1×D4 (registering the card fixer with runproc "under its run" when it is spawned by a detached runner that has no run): broken (F3).
- D1×D6 (the shared role constant against webster's own `verify-fixer` role): held for spawning, broken by D4's guard (F2).
- D2×D3 (an empty `fixer` value means no wrap and no wait-only wraps): held.
- D2×D5 (the wrapped forked closure, the lens-review-only skip, and each caller's edit scope staying inside what the wrapped gate checks): held for Darn-Write (verify fails on a dirty tree), Plan-Write and Discussion-Write (one entry each).
- D2×D7 (writers stop looping on their validator, so the gate fails into the fixer; the deadline finalize's fixer while the writer is mid-turn): broken (F1).
- D2×D8 (records keyed to the row visit against the `Expired` probe's "saved before the run's last attach"): held.
- D3×D4 (a gate key's `fixer` field against the scope of callers that hand off): broken (F4).
- D3×D6 (`fixer_timeout_min` and `recovery_timeout_min`): held.
- D4×D8 (the card record in the key directory against the fixer's own call keyed by its strand name, and a recovery strand's re-run on the same key): held.
- D4×D5 (card fixer makes no commit, and the re-run covers the uncommitted diff): held.
- D7×D4 (fork stencil text keyed on the hub's `card` key `fixer`): held.
- Constraint "594 s" against D3's 30-minute default: held. Go's re-run sits outside the fixer timeout.
- Constraint "after an attach only `Expired` probes run" against D2's probe: held.
- AC D2 "a finalize evaluation runs the fixer at most once" against D2's rationale: broken (F1).
- AC D1 "ending the run's processes stops a live fixer" against D2's restart path: held. A strand killed on resume reads dead through "strand gone".

**F1 — BLOCKING, design — D2 (deadline bound and Rationale), AC D2 "finalize evaluation runs the fixer at most once"**
The deadline finalize spawns a fixer while the writer is mid-turn, so two agents edit the worktree at once.
D2's Rationale rests on "the writer sits idle at its turn boundary, so the fixer edits a worktree nobody else is touching".
But D2's own deadline bound counts a fixer in "the deadline finalize after it runs the wrapped closure again".
That finalize comes right after a failed evaluation re-prompted the writer (budget left), so the writer is working its re-prompt.
The wait loop finalizes on the deadline whatever the writer's turn state, and `finalize` runs `evaluateGate(true)` before it removes the strand.
With D7 in force, every Plan-Write and Discussion-Write failure reaches this path.
Evidence: `internal/shuttleengine/wait.go:416-420` (deadline finalize with no idle check), `wait.go:1781-1792` (gate evaluated before the cleanup at 1830-1841), `gate.go:64` (attempts re-prompt a live writer).
Fix: a final evaluation off the writer's idle boundary spawns no fixer, or the writer is stopped first. The bound and the AC change to match.

**F2 — MEDIUM, consistency — D4 Bound (role guard) against D4 Rationale and D6**
D4's hand-off guard tests webster's `verify-fixer` role, so the gate verb must know webster's role.
That contradicts D4's own "the gate verb learns nothing of webster".
It also contradicts PATTERN-agent-name's "a spawned role is its module's constant": gatecli would repeat webster's literal or import websterengine.
D6 keeps `verify-fixer` as webster's role, and D1 says "a caller may name its own role", but no decision says where the full set of fixer roles lives.
Evidence: `internal/websterengine/verifyfixer.go:26` (`verifyFixerStrandRole`), `internal/gatecli/detachfront.go:233` (the front reads only the strand name), `internal/agentname/agentname.go:242` (`MatchesRole` takes one base).
Fix: name one owner of the fixer-role set, for example `gatefixer` exporting every fixer role with webster passing its role in, and have D4 test against that set.

**F3 — LOW, consistency — D1 (runproc registration and "a spawned strand whose run cannot be resolved … is stopped at once") against D4**
D1 registers every fixer "under its run" and stops one whose run cannot be resolved.
D4's fixer is spawned by a detached runner that belongs to no run.
`registerRunner` registers nothing for the prime or a worktree outside a hub, so neither decision says what "its run" means for a card fixer.
If "run" means the lym run, a card fixer outside a pair is always stopped at once.
Evidence: `internal/gatecli/detachfront.go:807-818` (`registerRunner` returns early outside a hub or with no slug).
Fix: D1 or D4 says the card fixer registers under the caller's pair slug, as the runner does, and what happens where there is none.

**F4 — MEDIUM, design — D3 (gate key `fixer` field) against Scope Out and D7**
D3 accepts `fixer` on any gate key and puts no limit on which keys may carry it.
A hub that sets it on `darn-write` makes the darn writer's own capped `lym gate test --gate darn-write` calls hand off to a fixer.
That contradicts Scope Out ("the darn writer's own `lym gate test` runs during its work") and D7 ("the darn writer stencil is unchanged").
It also puts a D4 card-style fixer beside D2's Darn-Write row fixer in the same worktree, which neither decision covers.
The attack-surface rule requires the decision that introduces the field to state its limit.
Evidence: `internal/gateslot/template.yaml` `gates:` block (keys that any agent call reaches), `internal/gatecli/detachfront.go:232-264` (any strand call is capped regardless of key).
Fix: D3 restricts the field to keys an agent strand calls outside a gated row, or refuses it on keys a row runs (`darn-write`, `drift-verify`, `code-burler-fix`). Alternatively, it states the limit and what it lets through.

Tool calls: none.
