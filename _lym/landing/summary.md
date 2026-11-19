# Hardener: autonomous behavior-review campaign; retire treadle, crucible

## What changes

A board task with `recipe: hardener` now runs an unattended behavior-review campaign against a live-substrate lym module and lands its fixes through Finalize.
The run walks Preflight, Record-Write and a Harden-Review ring, then Describe, PR-Review, Finalize and Friction-Reflect.
The task body is the campaign charter: the module, its code paths, the invariants to drive, and what is out of scope.
Each round is Harden-Bouncer, Harden-Burler-Review and Harden-Burler-Fix.
The reviewer drives the real substrate in throwaway local hubs, and Go independently re-verifies each fix with a `live` gate.
The campaign ends converged or at the round cap, and a capped or circling campaign escalates to its parent through the Bouncer's existing notice and decision verbs.

`internal/treadleengine`, its stencils and roles, the `Tenter` name and `crucible/` are deleted.
Their durable method moves into the `internal/hardenshed` package doc and the `harden-rubric-review` stencil.
Treadle had no consumer, and the operator's 2026-11-18 decision replaces the earlier one to keep its stencils for Hardener.

## Design

- **Recipe.**
  `hardener-recipe.yaml` is shaped like darn, with no Drift rows.
  `hardener` is admitted in every closed recipe list: `shedcli`, `shedrun`, batten's child recipes, `loomrecipe` and `reportengine`.
  The arm reads tree scope and verify attempts from `hardener.yaml`, never `darn.yaml`.
- **Round rows.**
  The ring reuses burler.
  The review row runs with tool use and source fix scope.
  `HardenFix` is the `BurlerFix` constructor with its gates fixed to `clean`, `verify` and `live`, so a recipe cannot turn the live gate off.
  Reviewer and fixer model lists expand cyclically to `hard_max_bounces`, because burler clamps a list to its last entry and adjacent rounds would otherwise share a model.
- **Loom contract.**
  `HardenFix` joins `shedrecipe.LoomBuildEngines`.
  A build row carrying a segment counts as lying at its segment's Bouncer.
  It is refused when no edge from that Bouncer reaches it, or when its segment holds no Bouncer.
  Recipes whose build rows carry no segment are checked as before.
- **Live gate.**
  It runs `harden-live` once, and a failure re-prompts the fixer up to `live_attempts` times.
  When they run out the row ends with the live report red and the failures deferred, instead of halting the run.
  With `llm_driving` false and `concurrency` above 1 it also runs `harden-live-copies --copies <n>`, whose failures go to the live report but never fail the gate.
  A live run that starts no test binary reads as red.
  It never runs tier 3.
- **`lym gate test --copies <n>`.**
  One lease of n times the tier's slot count replaces the tier run's own acquires, and n concurrent copies of the key's tier command run inside it.
  It needs `--gate`, refuses a key naming tier 3 or more than one tier, and refuses a call with no package.
  A lease above the pool is clamped and the clamp is reported.
- **Sandbox hubs.**
  `lym harden sandbox new|list|clear` (`internal/hardencli`) builds throwaway local hubs through `fabriccli.CloneAndWire` under a root in the worktree's `.lym`.
  The hubs have no GitHub remote and commit `selfreport: false` and `friction: ""`.
  `new` deploys `.dev-bin` only when it is not HEAD's build.
  Go reaps the root at the start and end of each round row, recording strays by row and reap point.
  The reap ends recorded processes through `runproc.End` and kills by the `/proc` scan only processes no record names.
  That scan is a new orphan reaper beside `lym tender reap`, and `PATTERN-child-processes` is amended for it.
  Finalize and the capped or circling halt reap too.
  `internal/pairteardown` runs `lym harden sandbox clear` before removing a worktree holding a root, and keeps the worktree with a way forward when the clear fails.
- **Handoff.**
  The Bouncer's facts carry each round's models, live outcome, stray report, deferred findings and fix commits.
  CONVERGED needs no BLOCKING or MEDIUM finding, a green live gate, and a reviewer model different from the previous round's.
  A Harden-Bouncer never auto-accepts a round whose live report is red.
  Each round is archived under `_lym/harden/round-<n>/`, and an approved settle archives the final ledger through the new `harden` commit seam.
- **`hardener.yaml`.**
  It holds the round cap (default 7), hard cap (10), model lists, judge, seed, `concurrency` (3), `live_attempts` (2), `tree_scope` and `verify_attempts`.
  Task `run_settings` may set `packages`, `llm_driving`, the caps, models and attempt counts.
  `auto_accept`, `reviewer_fan`, `fixer_fan` and `tree_scope` are Guard keys, refused at seed with the hub-file write as the way forward.
- **Preflight.**
  It refuses an empty or missing `packages`, a package with no test of a tier the live keys name, a `reviewer` list that puts one model in adjacent rounds, a `max_bounces` above `hard_max_bounces`, and a hub with no `harden-live` or `harden-live-copies` key of its own or a live key naming tier 3.
  Each refusal names its way forward.
- **Agents reinterpretation.**
  A `configreg.Reinterpretation` on `agents` strips the `treadle-judge` and `treadle-targeting` keys from `skills` and `efforts`, so a hub `agents.yaml` naming them still loads.

## Landing

The three hub gate values are operator writes to the hub's `gate.yaml`, made after this lands; no part of the change writes them.
Run from the hub:

```
lym config gate --layer default --set 'gates.harden-burler-fix.tiers=[0, 1]'
lym config gate --layer default --set 'gates.harden-live.tiers=[2]'
lym config gate --layer default --set 'gates.harden-live-copies.tiers=[2]'
```

Until they are set, Preflight refuses every `hardener` run on the hub, which is the intended refusal.

## To check by hand

- No live `hardener` campaign has been driven end to end against a real hub; the recipe, rows, gate and sandbox verbs are covered by unit and integration tests only.
- `archive-written-outside-anchor`: the round archive under `_lym/harden/` may be written outside its anchor.
- `archived-stray-report-unchecked`: the archived stray report is not verified after the copy.
- `bouncer-red-live-and-halt-reap-unpinned`: the Bouncer's red-live escalation and the halt reap are not pinned by a test.
- A round agent can still run `lym selfreport create` by hand, which files on the real upstream; the committed override stops only the automatic paths and the rubric forbids the call.
- The verify gate failed on `internal/gatecli` `TestRunTierCopies/RedCopyMakesTheCallRed` and `internal/shedbuild` `TestBuild_EveryRegisteredEngineBuilds/HardenFix`.
  Fix commit `8bdb302b17b7636a60f39a8166641ea91510f3bf` made the first race-free and covered `HardenFix` in the second; re-run both.
- The reaps derive socket keys and read process records with the running lym's own code, so a round that changes either can leave a server or process to the next reap.

## Merge danger

**Door:** one-way
**Blast radius:** hardener
