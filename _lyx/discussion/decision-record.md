# Decision record — lenses-module

## Goal

Every lens and fan lyx has lives in a config module of its own, `lenses`, with its file `lenses.yaml`, which burler's reviewers and fixers, loom's twill writers and darn's fan keys all read.
A hub's `lenses.yaml` holds every lens with its prose and every fan, reconcile keeps it complete, and the template's `twill-discussion` fan is the seven-lens fan the hub runs today as a trial.

## Scope

In:

- A new package `internal/lenses` that owns the `lenses` config module, its template being today's `internal/burlerengine/template.yaml` content.
- `burlerengine`, `burlercli`, `loomengine`, `loomcli` and `darnengine` read lenses and fans from `lenses` instead of `burlerengine`.
- configreg: `lenses` registered with reconcile that adds and keeps per name (D2); `burler` deregistered, since `lenses` and `fans` are the only keys `burler.yaml` holds, so `lyx config burler` gives way to `lyx config lenses`.
- Two declared key moves, `burler.lenses -> lenses.lenses` and `burler.fans -> lenses.fans`.
- The template's `twill-discussion` fan becomes `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `goal-scope`, `board-fit`, `leanness`.
- Every text that names `burler.yaml` as the home of lenses or fans now names `lenses.yaml`: the loom and darn template key comments, refusal messages, package docs and `docs/overview.md`.
  The template's header comment is rewritten to D2's reconcile and D3's load, since today's says the file is seed-only and a deleted lens stays deleted.

Out:

- Which fans loom and darn name, and their defaults in `loom.yaml` and `darn.yaml`.
- Lens prose other than the `twill-discussion` fan's membership; no lens is added or deleted.
- The research documents under `docs/research/`, which are measurement reports and keep naming `burler.yaml`.
- `writer-reviewer-lens-align`'s measurement of the seven-lens fan and its Plan, Webster and Merge halves, which stay open in that entry.
- The post-landing reconcile on the loomyard hub, which the orch runs after landing; D6 says what the code makes of it.

## Decisions

### D1 — `internal/lenses` owns the module

Decision: a new package `internal/lenses` holds the `lenses` module: the embedded template, the decode shape, the load, fan resolution and the per-round fan key grammar with its validation, all moved from `internal/burlerengine`.
It imports only the standard library, `configengine` and `gopkg.in/yaml.v3`, and the `PATTERN-leaf-packages` background lists it with that set.
`burlerengine` keeps its round logic and consumes the moved lens type; it holds no lens or fan data of its own.
The spec keeps `lenses` and `fans` as guard keys, so no local layer and no run setting holds them, and keeps the module per-worktree (not `HubWide`), as burler is today: its hub file is the repo default on the board.
Every refusal that told the operator to fix or define something in `burler.yaml` names `lenses.yaml` instead.
Bound: a run's `run_settings` holding a `burler` object is refused as naming an unregistered module, where today it is refused for holding guard keys.
Rationale: the operator decided lenses and fans leave burler; a leaf lets burler, loom and darn import it with no cycle and keeps it from growing a dependency on any of them.

### D2 — reconcile adds per name and keeps what the template lacks

Decision: `lenses` and `fans` are declared as per-name maps, a path kind `yamlengine` and `configsync` gain beside open maps:

- Reconcile adds each lens or fan the template holds and the file lacks, reported `added` per name, and keeps the file's prose and fan lists, a fan list being carried whole.
- A lens or fan the file holds and the template lacks is kept and reported in a result list of its own, never in `removed` and never pruned, at the base layer and at sparse layers alike.
- `lyx config lenses --set` knows any name under either path, so a lens or fan the template lacks is added there.

Bound: reconcile never deletes a lens or fan; an operator removes a hub-only entry by editing the file.
A lens the template drops stays in every hub file that holds it, reported on each reconcile, so a hub fan naming it keeps resolving.
Rationale: the operator asked reconcile to carry added and report dropped lenses and fans; reconcile cannot tell a lens the template dropped from one the hub added, and pruning would remove the config-only trial route that `writer-reviewer-lens-align` used and that `review-setup-audit` and `plan-writer-finding-classes` plan to use, and would break a loom or darn fan key naming a pruned lens at the next config load.

### D3 — load through `configengine.LoadOrTemplate`

Decision: `lenses` loads through `configengine.LoadOrTemplate` and then decodes the resolved bytes strictly.
An absent base file resolves the template, and a lens or fan the stack lacks resolves to its template value through the load's fill, so fan resolution reads the loaded config alone and drops burler's per-name template fallback.
Burler's own loader, `StackLayers` with no template fill, goes away, and the `PATTERN-config-strictness` background lists `lenses` among the degrading callers.
Bound: lens prose now passes through the load's environment expansion, as every module's values do; no template prose contains a `$` today.
Rationale: the template fill gives in one place the per-name fallback burler hand-rolls, and a deploy that adds a lens works before any reconcile.

### D4 — burler's file migrates through two key moves

Decision: `configreg.KeyMoves` gains `burler.lenses -> lenses.lenses` and `burler.fans -> lenses.fans`, after the existing moves.
At every layer a reconcile walks, the move runs before the modules are reconciled, as it does today: at the board's base layer an absent `lenses.yaml` starts from the template (or the prime's copy under seeding), each burler key's mapping replaces the template default whole, and on `--apply` the same pass's module reconcile then adds every template lens and fan the moved mappings lack.
`burler.yaml`, emptied by the two moves, is deleted by the move.
Bound: a target `lenses` or `fans` already holding a non-default value at a base layer, or any value at a sparse layer, is kept and the burler value dropped, as for every key move, and the drop is reported in the result's kept-target line; the move is whole-key, never per name.
A dry run writes nothing and reports the two moves, but not the additions, which the module loop reports only once `lenses.yaml` exists.
Rationale: the declared key move is the existing, tested route for a key changing modules at every layer, and its report lines show the operator what moved.

### D5 — the seven-lens `twill-discussion` is the template's

Decision: the template's `twill-discussion` fan becomes `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `goal-scope`, `board-fit`, `leanness`, in that order.
The lens `premises`, which no fan names after this, stays defined in the template.
Rationale: the operator made the trial the default; the moved hub value then equals the template's, so the hub override stops being a divergence.

### D6 — the hub migrates at the first start verb after deploy

Decision: the migration runs through `hubreconcile.Ensure`, with no route of its own.
The registry fingerprint changes with this task, so the first start verb after the deploy runs Ensure's walk, whose board reconcile performs D4's moves and D2's additions and commits the board with `lenses.yaml` written and `burler.yaml` deleted.
The burler move result is applied, so its path is among the files the walk hands the board commit; the plan makes that commit stage the deletion if it does not already.
The orch's post-landing `lyx config reconcile` then confirms that `lenses.yaml` holds the full set and that no `burler.yaml` remains.
Rationale: Ensure already owns reconciling a hub after a binary change (`PATTERN-binary-change-reconcile`).

## Constraints

- `PATTERN-config-layers`: only `configengine` reads, stacks and locates layer files.
- Docs land in the same commit: `internal/lenses/doc.go` documents the module, and `internal/burlerengine/doc.go`, `internal/burlercli/doc.go`, `internal/darnengine/doc.go` and `docs/overview.md` name `lenses.yaml`.

## Auto-mode assumptions

- The session ran autonomously; every pick is in the support log's question ledger.

## Open risks

- Whether the board commit's pathspec stages a tracked file's deletion; legacy config files removed by a hub-wide seed reach the commit the same way.
- A machine still on the old binary re-seeds `burler.yaml` at its next walk, and the new binary's next walk moves and deletes it again with burler's values dropped as kept-target; the board takes one commit per alternation until every machine has deployed, and no lens value is lost.

## Acceptance criteria

- A board config dir holding a `burler.yaml` of today's hub shape (the 2026-10-12 lenses and fans plus the seven-lens `twill-discussion` override) and no `lenses.yaml` reconciles with `--apply` into a `lenses.yaml` holding every template lens and fan, with burler's values kept, and no `burler.yaml`; the results report the two moves and the added lenses and fans.
- A dry run over the same dir writes nothing and reports the two moves.
- A `lenses.yaml` lacking a template lens or fan gains it on reconcile, reported added; one holding a lens or fan the template lacks keeps it and reports it in its own list, at the base layer and at a sparse layer; a tuned lens prose and a tuned fan list are kept unchanged.
- `lyx config lenses --set` adds a lens and a fan the template lacks, and a fan naming the new lens resolves.
- Loading with no `lenses.yaml` resolves every template fan; loading a `lenses.yaml` that lacks a template lens resolves a fan naming that lens with the template's prose.
- The moved fan-resolution tests keep covering the refusals of an unknown fan, an empty fan, a fan over the length cap and a fan naming an undefined lens.
- loom's and darn's fan keys, and loom's `discussion_lenses` and `plan_lenses`, resolve through `lenses.yaml`, and their refusals name `lenses.yaml`.
- The template's `twill-discussion` resolves to the seven lenses of D5, in order, with no hub override.
- `lyx config lenses` shows the resolved lenses and fans, and `burler` is no longer a config module.
- `hubreconcile.Ensure`'s walk over a hub in today's shape commits the board with `lenses.yaml` written and `burler.yaml` deleted.
- `internal/lenses` imports only its leaf set, checked by the leaf-package scan.

## PATTERN changes

- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only its closed import set, packages and sets named in the background.
- `PATTERN-config-strictness` — changes: Loading config: a caller adopts exactly one of strict `Load` or `LoadOrTemplate`, bar `runprofile`.

## Notes for the plan writer

- Both PATTERN entries keep their one-liners; the changes are to their background files, per D1 and D3.
- `configsync`'s key-move source side does no registry lookup, so `burler` can leave configreg while its file is still migrated.
