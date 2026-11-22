# Hardener writes its own charter and drives either hub layout

A hardener run now needs only a board task with `recipe: hardener` that names its modules.
The first run, harden-fabricengine, needed three hub gate writes, a task body written as the campaign charter, and a sandbox hub of a layout `lym harden sandbox new` could not build.
This change removes all three.

## What changes

- **The seed pass writes the charter.**
  The Harden-Bouncer's seed pass reads the modules `hardener.packages` names (code, verbs, docs, existing tests).
  When `packages` is empty it reads the task's title and labels, taken from the board entry.
  It writes the high-yield invariants to drive live with repro steps, out of scope, the cost declaration and the teardown check into round 1's focus.
  Each judge carries that focus forward and re-seeds it from the residual.
  The existing tests stay the fix gate's concern, never the campaign's subject.
- **The task body is an optional focus.**
  A body that states a focus binds the seeder: everything else in the module is out of scope, and the judge never steers a later round outside it.
  With no body the seeder takes the whole of the named modules.
- **`lym harden sandbox new --single-repo`** builds a one-repository hub from one local bare, code under `refs/heads/` and records under `refs/lym/`.
  The default stays the two-bare hub.
  The verb's output gains a `single_repo` field.
  `fabricengine` gains the one-bare seeder, and `fabriccli` a clone-and-wire helper for it.
- **No hub gate writes.**
  `gates.harden-live-copies` is retired.
  The concurrent copies run under `gates.harden-live`, which must name one tier below the billed tier.
  The template tiers of `harden-live` and `harden-burler-fix` pass Preflight on a hub with no hardener or harden gate config.
- **The rubric and stencils.**
  A new `harden-template-seed` stencil and a rewritten `harden-rubric-review` carry Crucible's live-driving rules:
  drive the real substrate, treat the focus list as a floor, improvise adversarial scenarios, and never treat cost as a reason not to drive.
  Other recipes' Bouncers keep the shared seed stencil unchanged.

## Design decisions

- No new row or step: the perch is a judge plus a burler, so the seed pass is the charter step and the review rounds are the burler half.
- Preflight reads the board entry only when `packages` is empty, and a missing entry is refused with a way forward naming `hardener.packages`, before any agent spends tokens.
- Folding the copies key into `harden-live` rather than deriving a tier keeps one key the hub can override.
- `RetireHardenLiveCopies` is registered as a `gate` reinterpretation.
  It removes a hub's `harden-live-copies` entry when it equals `harden-live`'s single tier below the billed tier.
  Otherwise it leaves the file alone and flags the way forward, which tells the operator to delete the retired key by hand.

## To check by hand

- Run `lym harden sandbox new --single-repo` in a hardener worktree and confirm the hub's code is on its branches and its records and board under `refs/lym/`.
- Run a hardener task with a title naming a module and no body on a hub with no hardener or harden gate config.
  Confirm it passes Preflight and that round 1's review file shows probes on a sandbox hub of each layout the seed picked.
- The `copies-key-retirement-no-reinterpretation` finding (design, blocking): confirm `internal/gateslot/reinterpret.go` covers every shape of a hub's `harden-live-copies` entry, including a hub with no `harden-live` entry.
- The `focus-filename-redeclared` finding (consistency, blocking): confirm the focus file's name is declared once in `internal/shedadapters/focus.go` and no other file repeats it.

## Merge danger

**Door:** one-way
**Blast radius:** hardener
