## Goal

```markdown
## Seen

The first hardener run, harden-fabricengine on 2026-11-21, needed three things from the operator before it could do any useful work:

- three hub gate writes (`harden-live`, `harden-live-copies`, `harden-burler-fix`), refused at Preflight until set, though each has a template value, and the copies key may name one tier only;
- a task body written as the campaign charter (module, code paths, invariants to drive, out of scope), since the recipe reads the body as the charter;
- sandbox hubs of the layout under test, but `lym harden sandbox new` builds only two-repository hubs (two local bares), so a campaign cannot drive a one-repository hub.

The run was cancelled after a few minutes.

## Design

A hardener run needs nothing beyond its task naming the module.

- **Hardener writes its own charter.**
  Before the first round, the campaign reads the module named by `packages` (or, absent that, the task's title and labels):
  its code, its verbs, its docs and its existing tests.
  It then writes the charter to the run directory: the code paths, the invariants to drive, the improvised scenarios and probes it will run on sandbox hubs, and what is out of scope.
  The existing tests are already run by the gate and are not the campaign's subject; the charter plans what they do not cover.
  A task body, when present, is a hint folded into the charter, never required.
  The judge's focus file keeps steering later rounds from the charter and what each round left standing.
- **Sandbox hubs of either layout.**
  `lym harden sandbox new` takes a layout flag and builds a one-repository hub from one local bare (code under `refs/heads/`, records and board under `refs/lym/`), as well as today's two-bare hub.
  The charter picks the layouts the module serves.
- **No hub gate writes.**
  The template tiers of the three harden gate keys suffice for Preflight.
  The copies run derives its single tier instead of needing a key of its own set by the hub, or the copies key is folded into `harden-live` with a copies count.
  The hub keeps the power to override.

## Done when

A board task with `recipe: hardener`, a title naming a module and no body, passes Preflight on a hub with no hardener or harden gate config, and its first round's review file shows improvised probes on a sandbox hub of each layout the charter picked.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-21): The charter step is Crucible's orchestrator fill

Decision: Model the charter step on Crucible, retired in 7346d6a8 and readable with git show 7346d6a8^:crucible/README.md, crucible/orchestrator-prompt.md and crucible/review-prompt-template.md. In Crucible the operator only said: do crucible on module X, read the crucible README. The orchestrator then read the module and filled every placeholder of review-prompt-template.md itself: what to read, the high-yield focus list of invariants to drive live with repro steps, out of scope, the live-substrate cost declaration and the teardown check. Hardener's charter step does that same fill before round 1, and the round prompt carries the template's live-driving rules: drive the real substrate directly, the focus list is a floor and the round improvises many more adversarial scenarios (crash and rebirth, cross-hub scope, dead-but-present state, mid-operation orphans, rapid churn), log as you go, never treat time or token cost as a reason not to drive. Rerunning the module's existing tests is the fix gate only, never the review. Carry over every rule of the template and README that Hardener's rubric and stencils lack.

Rationale: Operator, 2026-11-21: Crucible worked very well from a one-line ask; Hardener must do what Crucible did, and the existing tests are already run elsewhere.

### Added after Discussion (parent, 2026-11-21): The charter is the Harden-Bouncer seed pass, not a new step

Decision: This replaces the placement in the decision 'The charter step is Crucible's orchestrator fill'; its content rules stand. Add no new row or charter step. The perch is a judge plus burler by design, so Hardener reuses it: the Harden-Bouncer's seed pass is Crucible's orchestrator fill, and the review rounds are the burler half. Rewrite Hardener's seed so the seeder reads the module named by packages (code, verbs, docs, existing tests) and writes into its focus what Crucible's filled review-prompt-template held: what to read, the high-yield invariants to drive live with repro steps, out of scope, the cost declaration and the teardown check. The judge's focus file after each round is Crucible's re-seed: the residual to close, an enumeration sweep when findings circle, unreached regions once classes close, then a safety pass. Give the seed and judge calls of the hardener recipe their own stencil text or rubric where the shared Bouncer seed stencil cannot carry this, keeping other recipes' Bouncers unchanged. A task body is an optional hint to the seeder.

Rationale: Operator, 2026-11-21: the perch was designed as judge plus burler precisely so Hardener could reuse it; a hardener is an initial seeder, then the review rounds.

### Added after Discussion (parent, 2026-11-21): The operator can name the campaign's focus

Decision: This narrows 'A task body is an optional hint to the seeder' in the decision 'The charter is the Harden-Bouncer seed pass, not a new step'. The operator must be able to name what a campaign hardens: one module, several modules, or a small part of a module, such as the one-repository layout of fabric. packages names the module directories (one or more). The task body, when it states a focus, binds the seeder: the seeder derives the charter inside that focus and lists everything else in the module as out of scope; the judge never steers a later round outside it. With no body, the seeder takes the whole of the named modules. The one-line ask stays enough: modules plus an optional sentence of focus, never a hand-written charter.

Rationale: Operator, 2026-11-21: I must be able to say what to focus on, which module or modules, or a small part of a module.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
