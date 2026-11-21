# Hardener writes its own charter, drives either hub layout

`hardener-self-charter` · Task · enhancement, shed, burler, gate, stencil, fabric · running · Darn-Write

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
