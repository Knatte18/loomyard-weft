# Move lenses and fans into a `lenses` config module

Every lens and fan now lives in a config module of its own, `lenses`, with its file `lenses.yaml`.
Burler's reviewers and fixers, loom's twill writers and darn's fan keys all read it; `burler.yaml` no longer exists, and `lyx config burler` gives way to `lyx config lenses`.
Before, a hub's `burler.yaml` was a seed-only copy of the template from the day it was written, so lenses and fans added to the template since reached runs only through burler's hand-rolled per-name fallback and could not be tuned in the hub file.

No issue is closed.

## What changes

- New leaf package `internal/lenses` owns the template, the decode shape, the load, fan resolution and the per-round fan key grammar, all moved out of `internal/burlerengine`.
  It imports only the standard library, `configengine` and `gopkg.in/yaml.v3`; the leaf-package scan checks it.
- `burlerengine`, `burlercli`, `loomengine`, `loomcli` and `darnengine` read lenses and fans from `lenses`.
  Every refusal and template key comment that named `burler.yaml` as their home names `lenses.yaml`.
- The template's `twill-discussion` fan is now `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `goal-scope`, `board-fit`, `leanness`, so the hub's seven-lens trial becomes the default.
  The lens `premises` stays defined, though no fan names it.
- `burler` is deregistered from `configreg`.
  A run's `run_settings` holding a `burler` object, empty or not, is now refused as naming an unregistered module.

## Design decisions

- **Per-name maps.**
  `yamlengine` and `configsync` gain a per-name map kind, declared by `configengine.Spec.PerNameMaps` (and `Module.PerNameMaps` in `configreg`, covered by the registry fingerprint).
  Reconcile adds each template lens or fan the file lacks, reported `added` per name, and keeps the file's prose and fan lists.
  A lens or fan the template lacks is kept and reported in its own `KeptExtra` list, never pruned, at the base layer and at sparse layers.
  The load's unknown-keys warning, `Set`'s unknown-key check and the sparse prune honour the same kind, so a hub-only lens or fan is not warned about.
- **Consequence of keeping.**
  A later template edit to a lens's prose or a fan's membership does not reach a hub whose file holds that name; the operator deletes the entry so the next reconcile adds the template's, or edits it in.
  A template lens or fan cannot be removed from a hub; a fan is turned off at the fan key entry that names it.
- **`--set`.**
  `lyx config lenses --set` takes `lenses.<name>` as a string scalar and `fans.<name>` as a flow list, adds names the template lacks, and refuses a further dotted segment.
  `CheckLoad` resolves every fan in name order, so a `--set` leaving a fan empty, over the length cap or naming an undefined lens is refused and rolled back.
- **Load.**
  `lenses` loads through `configengine.LoadOrTemplate`, so a lens or fan the stack lacks resolves to its template value and burler's `StackLayers` loader and per-name fallback are gone.
  Lens prose passes through `${env:...}` expansion like every module's values.
- **Migration.**
  `configreg.KeyMoves` gains `burler.lenses -> lenses.lenses` and `burler.fans -> lenses.fans`, with a new per-move `KeepTarget` opt-out of `configsync`'s template-default overwrite.
  The moves never overwrite a present `lenses.yaml`: its `lenses` and `fans` are kept whatever their value, and burler's are dropped and reported as kept-target.
  A `lenses.yaml` started from the template in the same pass takes burler's mappings whole, and the module reconcile then adds the template entries they lack.
  Under seeding, an absent `lenses.yaml` starts from the prime's copy when it holds one, and burler's keys are dropped.
  The emptied `burler.yaml` is deleted, and `hubreconcile.Ensure`'s board commit stages that deletion, so the first start verb after deploy migrates the hub with no route of its own.
- **Docs and PATTERN.**
  `internal/lenses/doc.go` documents the module; the `burlerengine`, `burlercli`, `darnengine` and `yamlengine` docs and `configengine/doc.go` follow.
  `PATTERN-leaf-packages` lists `lenses` with its import set, and `PATTERN-config-strictness` lists `lenses` among the degrading callers.
  `docs/research/` keeps naming `burler.yaml`, being measurement reports.

## Check by hand

- After landing, run `lyx config reconcile --layer default --apply` on the loomyard hub (the orch's post-landing step) and confirm `lenses.yaml` holds the full set and no `burler.yaml` remains.
- A machine still on the old binary re-seeds `burler.yaml` at its next walk, and the new binary's next walk deletes it again with burler's values dropped as kept-target, leaving `lenses.yaml` unchanged.
  Bolt takes one commit per alternation until every machine has deployed, and the orch's check can see a re-seeded `burler.yaml` until the new binary's next walk.
- A `lenses.yaml` written before the first walk after deploy, such as by `lyx config lenses --set` or `--edit` scaffolding it from the template, makes that walk drop burler's tuned prose and fans as kept-target.
  A prime worktree-layer `lenses.yaml` written in that window does the same through the seeded start, even a sparse copy holding one fan.
  Run a start verb or `lyx config reconcile --layer default --apply` from the prime first.
- The dry run reports the two moves but not the additions, which the module loop reports only once `lenses.yaml` exists; check that the dry-run report matches this wording and the migration decision above.
- The gate's integration suite runs at the verify step after this description; this description did not run it.
