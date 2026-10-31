```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths, over the record pair `decision-record.md` and `support-log.md`.

**Paths traced, per decision**

- D1: `shedbuild` parse of top-level keys gains `contract` and `verify_source`. Survives. Only the parser decodes them.
- D2: batten's Run-Shed starts pawl with `lym loom start` (`internal/battencli/wire.go:525-539`). Survives under the pawl exception.
- D3: `peers.go:88-90` maps retired rows of both recipes, and `peers.go:247` reads rows from `tapestry` only. Breaks today, and D3 replaces it. Survives.
- D4, run parked on Preflight: resumes into Record-Write. Survives.
- D4, run parked on Drift-Baseline or Drift-Await: resumes past Record-Write with `subject: decision-record` and no record on disk. Breaks (F2).
- D4, run parked on Darn-Write: the halt comes only after a full writer session and the gate's re-prompts. Breaks the bound (F2).
- D4, `decision add` before Record-Write: refused by the existing missing-record refusal. Survives.
- D5: `validate.go:70-92` picks the validator by whether `DiscussionHistory` is wired, then commits through `CommitDiscussion` on a darn run. Survives if the validator choice is rekeyed. The darn-only commit stays harmless.
- D6: the Darn-Write `Commit` seam (`darnwiring.go:153-154`) commits the landing directory into the weft repo, not onto the task branch. "No commit past the fork" can still be read on warp. Survives.
- D7: `commitDarnDiscussion` is the `CommitDiscussion` seam of darn (`darnwiring.go:187-193`, `:245`), reached by `decision add` and `validate-discussion`. Removing the snapshot there covers both verbs. Pawl writes its snapshot through its own seam (`pawlwiring.go:134-138`). Survives.
- D10: `CheckSeed` has one caller, Tapestry-Preflight. Removing it leaves no other reader. Survives.
- D12, run settings and profiles: `moveRunSettings` translates them on read, and `LayerRunSettings` prepares the profile and the task separately and then merges them. Survives.
- D12, plan perch budget: the hard-cap-against-budget check and the plan budget fed to `recipeRouting` read `PlanMaxBounces` from `loom.yaml` (`loomengine/config.go:446`, `:1341`; `arm.go:224`). Breaks (F3).
- D13, child driver: `childDriverKind` and `arbiterstop/holdreading.go` call `RecipeOverride`. With the override gone they read `shed.yaml` plus the local layer that Seed-Child writes before `ChildDriver` resolves (`wire.go:1307-1319`). The `shed` keys are local-allowed (`sheddriver/config.go:77-84`), so a profile `shed` block reaches the child. Survives.
- D13, refusing the removed keys in run settings and profiles: the existing retired-key route is `RetiredKeys` with `CategoryNone`, which admits and drops them silently. Breaks (F1).
- D13, worktrees already seeded with a local `loom.yaml` or `darn.yaml` holding these keys: the overrides are dropped or refused, and the record says nothing about these worktrees. Breaks (F1).
- D13, `KeyCopies` from loom to darn for the arbiter keys (`configreg/keymove.go:51-57`): it copies a retired key into a retired key. Breaks, with low impact (F4).
- D13, loom's `migrateShedOverrides` Migrate (`loomengine/config.go:714-778`) still rewrites the retired keys. Breaks, with low impact (F4).

**Findings**

- **F1 — MEDIUM, design, D13 (decision-record.md, D13).**
  The existing retired-key route would let the removed keys through silently, not refuse them, and D13 does not name another route.
  - In `internal/configengine/doc.go:75-77`, a key in `RetiredKeys` keeps the category it had as a template key. These keys had `CategoryNone` (`loomengine/config.go:557-563`), and that category is admitted. Every load then drops retired keys.
  - So `PrepareRunSettings`, through `checkLocalDocument` (`configreg/runsettings.go:45-88`), would accept `loom.driver_kind` and drop it. That is the "silently running another" outcome D13 says it prevents.
  - The same route drops these keys from the local `loom.yaml` or `darn.yaml` that Seed-Child already wrote into worktrees seeded before landing. Those runs lose their tackler and arbiter overrides with no refusal. D13 and Open risks cover only `profiles.yaml` and task `run_settings`.
  - Evidence: `internal/configengine/doc.go:75-77`, `internal/loomengine/config.go:557-563`, `internal/configreg/runsettings.go:45-88`.
- **F2 — MEDIUM, design, D4 (decision-record.md, D4, paragraph on in-flight runs).**
  The handling of in-flight darn runs covers only runs that halt at Darn-Write's gate, and it understates what happens before that halt.
  - A darn run parked on Drift-Baseline or Drift-Await sits after Record-Write's new place, so it never passes Record-Write. With `subject: decision-record`, its Drift agent is told to read a record that does not exist (`driftengine/seats.go:62-63`).
  - A run that reaches Darn-Write with no record halts only after the writer's full LLM session. The writer is told to read the absent record as its scope and never to write it. The `discussion` gate then re-prompts that writer up to 3 times to fix a record it may not write (`darn-recipe.yaml`, Darn-Write `attempts: 3`).
  - D4 claims the run "halts at Darn-Write's discussion gate", but says nothing of the Drift rows or of the session spent before the halt.
  - Evidence: `internal/driftengine/seats.go:62-63`, `contracts/recipes/darn-recipe.yaml` (rows Drift-Baseline and Darn-Write).
- **F3 — MEDIUM, design, D12 (decision-record.md, D12).**
  D12 says what falls back across modules but not where the plan perch's budget is resolved and validated once its keys sit in `tapestry.yaml`.
  - Today loom's `LoadConfig` refuses a perch whose resolved hard cap is below its resolved budget. It does this from one `Config` that holds `PlanMaxBounces`, `ReviewMaxBounces` and the hard caps (`loomengine/config.go:4`, `:446`, `:1341`).
  - `recipeRouting` applies `loom.yaml`'s budgets (`internal/loomcli/arm.go:224`).
  - After the split, the plan budget and plan hard cap are in `tapestry.yaml`, while the review defaults they fall back to stay in `loom.yaml`. A per-module load or local check can no longer see both. A `tapestry.yaml` with `plan_hard_max_bounces` below the inherited `loom.review_max_bounces` would then pass, or the invariant is silently lost.
  - Evidence: `internal/loomengine/config.go:4`, `:446`, `:1341`, and `internal/loomcli/arm.go:224`.
- **F4 — LOW, design, D13 (decision-record.md, D13).**
  Two existing reconcile and migration paths write the keys D13 retires, and the record says nothing about either.
  - `KeyCopies` copies `loom.arbiter*` into `darn.arbiter*` (`internal/configreg/keymove.go:51-57`), so a reconcile could write a retired key into `darn.yaml`.
  - loom's `migrateShedOverrides` Migrate still rewrites `driver_kind` and the tackler keys (`internal/loomengine/config.go:714-778`), and `PrepareRunSettings` runs it on run settings.
  - Both become dead or conflicting paths unless D13 retires them too.
  - Evidence: `internal/configreg/keymove.go:51-57`, `internal/loomengine/config.go:714-778`.

**Tool calls**

1. `sheddriver/config.go` key categories and the drift subject handling: to check that a profile `shed` block is local-allowed, and what a decision-record subject does with a missing record. Not in my context.
2. `loomcli/validate.go:70-100`, the `childDriverKind` callers, and the pawl snapshot wiring: to check how `validate-discussion` picks darn and commits, and that pawl's snapshot path is separate. Not in my context.
3. `battencli/wire.go:1300-1345`: to check when `ChildDriver` resolves relative to Seed-Child's local-layer write. Not in my context.
