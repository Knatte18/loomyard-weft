```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guard, guarantee and claim about existing code in the decision record. Three claims break and three bounds have gaps.

**Claims attacked:**
- D3 "burler's lists clamp to their last entry": **held**. `internal/burlerengine/models.go:18-31` `pickRound` clamps to `len(choices)`. The record cites no file for it.
- D3/D4 "a recipe cannot configure the live gate off": **held** on a review with no findings. Burler runs the round's gates first and spawns the fixer only when they fail (`internal/burlerengine/prompt.go:231-242`), so the live gate still runs on a converging round.
- D4 "the stray check is the one `tmuxkit.Main` makes": **held** (`internal/testkit/tmuxkit/tmuxkit.go` package doc: the `/proc` scan, the socket check and the kill).
- D4 `--copies` "runs one copy per held slot and records the shortfall when the pool grants fewer": **broken** (F2).
- D5 "`selfreport: false` in the committed `loom.yaml` stops the automatic paths": **broken** (F3).
- D5 "the reaps touch only reed servers whose socket key derives from a hub path under the root": **bound gap** (F5).
- D5 "the verbs refuse outside a `hardener` run worktree": **held** for the verbs themselves. The reap bound does not reach hubs made outside the root (F6).
- D7 "a halted campaign keeps its last ledger only in the run directory": **held**, and the record states it.
- D8 "`internal/agentconfig` refuses no unknown role": **broken** (F1).
- D10 "every recipe whose build row carries none is checked as today": **held**. No existing build row carries a segment (`contracts/recipes/darn-recipe.yaml:110-113`, `contracts/recipes/tapestry-recipe.yaml:339-342`).
- D10 "the order clause places a segmented build row at its Bouncer": **bound gap** (F4).
- Constraints on `CloneAndWire`, `deploy-dev`, the lymtemp sweep and the Bouncer seam firing only on an approved settle: **held**. Evidence: the `hubforge` doc, `docs/sandbox-howto.md:10,66` and `internal/shedrecipe/entries_bouncer.go:118-139`.

**Findings:**

- **F1 — BLOCKING, design, D8 and its acceptance criterion ("agents.yaml naming `treadle-judge` still loads").**
  - D8's premise is false. `agentconfig.Load` "refuses a role key outside the roles lym starts", and `skillLists` returns `unusable` for an unknown key. A test row pins this.
  - Counterexample: a hub `agents.yaml` that tunes `skills: {treadle-judge: [...]}`. Once D8 removes the role, `Load` fails with `ErrInvalid`. This is a meaning change, so PATTERN-config-layers requires a `configreg.Reinterpretation` applied by `hubreconcile`. D8 says none ships.
  - Evidence: `internal/agentconfig/config.go:129-130,160-166`; `internal/agentconfig/config_test.go:89,97`.

- **F2 — MEDIUM, design, D4 `--copies` bound.**
  - A multi-slot acquire never takes a partial grant. It waits under the drain lock until it holds every slot it asked for, and clamps only a count above the pool's size.
  - A nested gate run "runs inside its parent's slot without acquiring, whatever slots its own tier asks for".
  - D4 runs the live gate "in held gate slots" and has it call `lym gate test --copies <concurrency>`. That call is nested, so its copies get only the parent's hold, typically one slot as a non-tier holder.
  - Result: the concurrent amplifier collapses to serial every round. The shortfall bound describes a grant that never happens.
  - Evidence: `internal/gateslot/doc.go:20-27,58-63`; `internal/gatecli/cli.go` `Long` ("Inside a slot an enclosing gate run holds it acquires no other").

- **F3 — LOW, consistency, D5 and its acceptance criterion (`selfreport: false`).**
  - `loom.yaml`'s `selfreport` key is retired and ignored; a config test row is titled "retired selfreport key ignored".
  - The only automatic filing path is the friction reflection, which files through `lym selfreport create`. `friction: ""` turns that off.
  - So the override works through `friction: ""` alone. The `selfreport: false` half, and the criterion that tests it, pin a key that does nothing.
  - Evidence: `internal/loomengine/config_test.go:211-213`; `internal/loomengine/template.yaml:37`; `internal/friction/doc.go:49`.

- **F4 — MEDIUM, design, D10 bound.**
  - The order clause checks only the indices of the record row and the build row on the on_done path. D10 substitutes the segment's Bouncer for the build row.
  - Counterexample: a `HardenFix` row whose segment's Bouncer lies after Record-Write but which no `on_stuck` edge reaches, or whose segment holds no Bouncer at all. The first passes the contract and builds nothing; for the second, D10 states no outcome.
  - The bound states which recipes it exempts, not what the loosened clause now lets through.
  - Evidence: `internal/shedbuild/loomcontract.go:34-44`.

- **F5 — LOW, design, D5 reap bound.**
  - Socket keys are a hash of the hub path, so a reap can derive one only from a hub directory that still exists under the root.
  - Counterexample: a round agent removes a hub by hand, or a row crashes after a hub is deleted but before its server dies. That reed server survives every later reap, and the stray report never sees it.
  - D5 states only the derivation-change gap.
  - Evidence: `internal/reedengine/server.go:33-36`.

- **F6 — LOW, design, D5 "zero stray state" claim.**
  - The verbs' refusal and the reaps bound only what lies under the root.
  - A round agent driving the `.dev-bin` lym can make a hub or pair outside the root with other lym verbs. Neither the reaps nor the stray report cover it, and the record states no bound for that case.
  - Evidence: D5 Bounds paragraph, `_lym/discussion/decision-record.md:101`.

**Tool calls:**
1. `grep` of `internal/gateslot/doc.go` and the recipe build rows: nested-holder semantics and build-row segments were not in context.
2. Read `internal/gateslot/doc.go:20-75` and the build rows: the partial-grant and inheritance rules were not in context.
3. Read the darn and tapestry build rows and the `friction` template line: whether existing build rows carry a segment was not in context.
4. `grep` of burler's empty-review handling: whether gates run when the fixer has no findings was not in context.
5. The first `grep` of call 1 also checked the `holding one slot` wording in the gateslot doc.
