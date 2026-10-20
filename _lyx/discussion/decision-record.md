# Decision record — lenses-module

## Goal

Every lens and fan lyx has lives in a config module of its own, `lenses`, with its file `lenses.yaml`, which burler's reviewers and fixers, loom's twill writers and darn's fan keys all read.
A hub's `lenses.yaml` holds every lens with its prose and every fan, reconcile keeps it complete, and the template's `twill-discussion` fan is the seven-lens fan the hub runs today as a trial.

## Scope

In:

- A new package `internal/lenses` that owns the `lenses` config module, its template being today's `internal/burlerengine/template.yaml` content.
- `burlerengine`, `burlercli`, `loomengine`, `loomcli` and `darnengine` read lenses and fans from `lenses` instead of `burlerengine`.
- configreg: `lenses` registered with reconcile that adds and keeps per name (D2); `burler` deregistered, since `lenses` and `fans` are the only keys `burler.yaml` holds, so `lyx config burler` gives way to `lyx config lenses`.
- Two declared key moves, `burler.lenses -> lenses.lenses` and `burler.fans -> lenses.fans`, and a per-move opt-out of `configsync`'s template-default overwrite that both set (D4).
- The template's `twill-discussion` fan becomes `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `goal-scope`, `board-fit`, `leanness`.
- Every text that names `burler.yaml` as the home of lenses or fans now names `lenses.yaml`: the loom and darn template key comments, refusal messages, package docs and `docs/overview.md`.
  The template's header comment is rewritten to D2's reconcile and D3's load, since today's says the file is seed-only and a deleted lens stays deleted.
  It says that reconcile keeps every present lens and fan, so a later template edit reaches the file only once its entry is deleted or edited in, and that a template lens or fan cannot be removed, a fan being turned off at the fan key entry that names it.

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
An empty `burler` object, dropped today, is refused as well, since the module lookup runs before the empty-object skip.
Rationale: the operator decided lenses and fans leave burler; a leaf lets burler, loom and darn import it with no cycle and keeps it from growing a dependency on any of them.

### D2 — reconcile adds per name and keeps what the template lacks

Decision: `lenses` and `fans` are declared as per-name maps, a path kind `yamlengine` and `configsync` gain beside open maps:

- Reconcile adds each lens or fan the template holds and the file lacks, reported `added` per name, and keeps the file's prose and fan lists, a fan list being carried whole.
- A lens or fan the file holds and the template lacks is kept and reported in a result list of its own, never in `removed` and never pruned, at the base layer and at sparse layers alike.
- `lyx config lenses --set` knows any name under either path, so a lens or fan the template lacks is added there.

The per-name kind is declared on the `lenses` module's `configengine.Spec` and honoured by every template comparison that spec drives: the load's fill and unknown-keys warning, `Set`'s unknown-key check and the sparse-layer prune.
So a hub-only lens or fan is never warned as a key no loader reads.

Bound: reconcile never deletes a lens or fan; an operator removes a hub-only entry by editing the file.
A lens the template drops stays in every hub file that holds it, reported on each reconcile, so a hub fan naming it keeps resolving.
Reconcile keeps the file's value for every present lens and fan, so a later template edit to a lens's prose or a fan's membership never reaches a hub whose file holds that name.
An operator or a landing task takes such a template change by deleting the entry from the hub file, so the next reconcile adds the template's, or by editing it in.
A lens or fan the template holds cannot be removed from a hub: deleting it from the file makes the next load resolve the template's value and the next reconcile write it back, reported added.
An operator turns a fan off by pointing the fan key entry that names it at another fan, or by setting that entry to `""`, which runs the round solo or with the single fixer.
`--set` on a lens or fan writes the default or worktree layer only, the local layer refusing both guard keys.
A name the template holds is a tune of it, and a lens or fan name is one dot-free YAML key.
`--set` takes `lenses.<name>` as a string scalar and `fans.<name>` as a YAML flow list of lens names, such as `[a, b]`, that replaces the fan whole, and refuses a key with a further dotted segment under either path as unknown through `Set`'s unknown-key check.
The `lenses` module's `CheckLoad` resolves every fan after the load, in name order, and reports every failure, so its error text is deterministic for `--set`'s before-and-after comparison and a `--set` leaving a fan empty, over the length cap or naming an undefined lens is refused and rolled back.
Rationale: the operator asked reconcile to carry added and report dropped lenses and fans; reconcile cannot tell a lens the template dropped from one the hub added, and pruning would remove the config-only trial route that `writer-reviewer-lens-align` used and that `review-setup-audit` and `plan-writer-finding-classes` plan to use, and would break a loom or darn fan key naming a pruned lens at the next config load.

### D3 — load through `configengine.LoadOrTemplate`

Decision: `lenses` loads through `configengine.LoadOrTemplate` and then decodes the resolved bytes strictly.
An absent base file resolves the template, and a lens or fan the stack lacks resolves to its template value through the load's fill, so fan resolution reads the loaded config alone and drops burler's per-name template fallback.
Burler's own loader, `StackLayers` with no template fill, goes away, and the `PATTERN-config-strictness` background lists `lenses` among the degrading callers.
Bound: lens prose, template and hub-held alike, passes through the load's `${env:...}` expansion, as every module's values do.
A bare `$` is untouched, and a `${env:NAME}` marker in prose expands, or fails the load when `NAME` is unset and no default is given; no template prose holds such a marker.
Rationale: the template fill gives in one place the per-name fallback burler hand-rolls, and a deploy that adds a lens works before any reconcile.

### D4 — burler's file migrates through two key moves

Decision: `configreg.KeyMoves` gains `burler.lenses -> lenses.lenses` and `burler.fans -> lenses.fans`, after the existing moves.
At every layer a reconcile walks, the move runs before the modules are reconciled, as it does today: at the board's base layer an absent `lenses.yaml` started from the template takes each burler key's mapping, which replaces the template default whole, and on `--apply` the same pass's module reconcile then adds every template lens and fan the moved mappings lack.
Under seeding, an absent `lenses.yaml` starts instead from the prime's copy when the prime holds one; that copy keeps whatever `lenses` or `fans` it holds, and burler's are dropped as kept-target, as for a present target.
`burler.yaml`, emptied by the two moves, is deleted by the move.
Bound: these two moves never overwrite a present `lenses.yaml`.
At a base layer where `lenses.yaml` exists, its `lenses` and `fans` are kept whatever their value, even one equal to the template default, and burler's are dropped and reported as kept-target; at a sparse layer any target value is kept the same way.
A target takes a burler key's mapping only when it was started from the template in the same pass or does not hold that key; the move is whole-key, never per name.
So a board `lenses.yaml` written before the first walk after deploy, for example by `lyx config lenses --set` or `--edit` scaffolding it from the template, makes that walk drop burler's tuned prose and fans as kept-target.
A prime worktree-layer `lenses.yaml` written in the same window, for example by `lyx config lenses --set` at the worktree layer, does the same through the seeded start, so even a sparse copy holding one fan drops burler's whole `fans`.
For both, the way forward is a start verb or `lyx config reconcile --layer default --apply` from the prime before the first `lenses.yaml` write.
The kept-target line for these two moves names the dropped and kept keys without rendering their mappings.
A base layer's dropped values stay in Bolt's history, in the parent of the commit that deletes `burler.yaml`; a sparse layer's are kept by nothing.
A dry run writes nothing and reports the two moves, but not the additions, which the module loop reports only once `lenses.yaml` exists.
Rationale: the declared key move is the existing, tested route for a key changing modules at every layer, and its report lines show the operator what moved.
The general template-default overwrite would let an old binary's re-seeded `burler.yaml` revert a migrated `lenses.yaml` that equals the template, `twill-discussion` included, so these two moves give it up.

### D5 — the seven-lens `twill-discussion` is the template's

Decision: the template's `twill-discussion` fan becomes `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `goal-scope`, `board-fit`, `leanness`, in that order.
The lens `premises`, which no fan names after this, stays defined in the template.
Rationale: the operator made the trial the default; the moved hub value then equals the template's, so the hub override stops being a divergence.

### D6 — the hub migrates at the first start verb after deploy

Decision: the migration runs through `hubreconcile.Ensure`, with no route of its own.
The registry fingerprint changes with this task, so the first start verb after the deploy runs Ensure's walk, whose board reconcile performs D4's moves and D2's additions and commits the board with `lenses.yaml` written and `burler.yaml` deleted.
The burler move result is applied, so its path is among the files the walk hands the board commit, and the board commit stages the deletion of a written path that no longer exists, as legacy-file removals already rely on.
The orch's post-landing `lyx config reconcile` then confirms that `lenses.yaml` holds the full set and that no `burler.yaml` remains.
A `burler.yaml` an old binary re-seeded since (Open risks) is removed by that reconcile with `--apply`, through the same moves.
Rationale: Ensure already owns reconciling a hub after a binary change (`PATTERN-binary-change-reconcile`).

## Constraints

- `PATTERN-config-layers`: only `configengine` reads, stacks and locates layer files.
- Docs land in the same commit: `internal/lenses/doc.go` documents the module, and `internal/burlerengine/doc.go`, `internal/burlercli/doc.go`, `internal/darnengine/doc.go` and `docs/overview.md` name `lenses.yaml`.

## Auto-mode assumptions

- The session ran autonomously; every pick is in the support log's question ledger.

## Open risks

- A machine still on the old binary re-seeds `burler.yaml` from its old template at its next walk, and the new binary's next walk deletes it again with burler's values dropped as kept-target under D4's bound, leaving `lenses.yaml` unchanged; the board takes one commit per alternation until every machine has deployed.
  An operator edit made to `burler.yaml` through an old binary after migration is dropped the same way.
  A re-seeded `burler.yaml` stays on Bolt until the new binary's next walk, which runs only at its next build change or an absent repo default, so the orch's post-landing check can see one.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - prime-copy-start-drops-burler-mappings (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A board config dir holding a `burler.yaml` of today's hub shape (the 2026-10-12 lenses and fans plus the seven-lens `twill-discussion` override) and no `lenses.yaml`, with no prime copy of `lenses.yaml` to seed from, reconciles with `--apply` into a `lenses.yaml` holding every template lens and fan, with burler's values kept, and no `burler.yaml`; the results report the two moves and the added lenses and fans.
- A dry run over the same dir writes nothing and reports the two moves.
- A board config dir holding a migrated `lenses.yaml` equal to the template plus an old-template `burler.yaml` reconciles to an unchanged `lenses.yaml`, with `burler.yaml` deleted and the drop reported as kept-target.
- A board config dir holding a non-default `lenses.yaml` and a `burler.yaml` keeps `lenses.yaml` unchanged, deletes `burler.yaml` and reports the drop as kept-target, and a sparse layer does the same for any target value.
- A board config dir holding a `burler.yaml` and no `lenses.yaml`, reconciled with seeding from a prime whose worktree-layer `lenses.yaml` holds a `fans` mapping, writes a `lenses.yaml` keeping the prime's `fans`, deletes `burler.yaml` and reports burler's `fans` dropped as kept-target.
- Run settings holding a `burler` object, empty or not, are refused as naming an unregistered module.
- A `lenses.yaml` lacking a template lens or fan gains it on reconcile, reported added; one holding a lens or fan the template lacks keeps it and reports it in its own list, at the base layer and at a sparse layer; a tuned lens prose and a tuned fan list are kept unchanged.
- `lyx config lenses --set` adds a lens and a fan the template lacks, and a fan naming the new lens resolves; a `--set` leaving a fan empty, over the length cap or naming an undefined lens is refused and leaves the file unchanged.
- Loading a `lenses.yaml` holding a hub-only lens logs no unknown-keys warning.
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
