## Goal

A hub's lenses resolve from a language-neutral base template plus a predefined language layer the hub picks, so a C# hub such as NORCE Models no longer starts with lenses worded for this Go repository.
A hub can still add and override lenses and fans by name, and can now remove a lens it does not want.

## Scope

In:

- A language-neutral base `internal/lenses/template.yaml`.
- One embedded language layer per language beside it: `go`, `csharp` and `python`.
- A new `lens_set` key in `lenses.yaml` that picks the layer, and a new `remove` key that drops lenses.
- A template overlay in `internal/configengine` (D4).
- The docs that describe the lens library and template judging: `internal/lenses/doc.go`, the template's header comment, `internal/configengine/doc.go` and `internal/configsync`'s package doc.

Out:

- Detecting a hub's language from its files: lyx carries no language logic in code, and the operator picks the layer.
- Writing `lens_set` into any hub's `lenses.yaml` on Bolt: the task worktree cannot write the board's config (D6).
- Rewriting the header comment that existing hub files copied from the old template: it is the operator's text.
- The `language` config module (`internal/languageconfig`): it keeps naming no language.
- Changing any review rubric or stencil wording.

## Decisions

### D1 — The base template is language-neutral

`internal/lenses/template.yaml` keeps every lens and fan name it has today, and its prose names no language.
The lenses worded for Go today are reworded neutrally in the base: `error-handling`, `test-gaps`, `performance`, `api-design` and `concurrency`.
`pattern-fit`, `code-pattern-fit` and `docs-consistency` stay in the base and are reworded to read the repository's own rules where it keeps them: the PATTERN entries and `pattern/` files when the repository has a `PATTERN.md`, otherwise the rules its `CLAUDE.md` files state.
None of the five reworded lenses gets a `formerTemplateValues` entry in configsync: the `go` layer carries their old prose, and stripping a hub's pinned copy of it would change that hub's loaded value under another layer.

Rationale: where a hub keeps its rules is a property of the hub, not of its language, so the rules lenses belong in the base.
A hub that keeps its rules elsewhere overrides the lens by name, as it can today.

### D2 — One embedded language layer per language

Each layer is one YAML file under `internal/lenses/sets/`, named `<name>.yaml`, embedded into the binary beside the base template.
A layer holds only entries of the two per-name maps, `lenses` and `fans`: each entry replaces the base entry of the same name whole, or adds a name the base lacks.
The shipped layers:

- `go` holds today's wording of the five lenses D1 neutralizes, verbatim.
- `csharp` holds C# wording of the five lenses (nullable reference types and `IDisposable`/`using` in `error-handling`, `async`/`await`, `Task` and thread-safety in `concurrency`, xUnit/NUnit-style coverage in `test-gaps`, allocation and LINQ cost in `performance`, public-surface and assembly compatibility in `api-design`).
  It adds a `numerics` lens on numerical stability and units (floating-point comparison, overflow and loss of precision, unit consistency, out-of-range physical values), and overrides the `review-code` and `full` fans to add `numerics` after their base lenses.
  Its `error-handling` and `numerics` prose defer to the repository's own rules on guards and exception handling, so a repository that forbids argument guards or try/catch around its own code, as NORCE Models does, gets no finding asking for one.
- `python` holds Python wording of the five lenses (exception handling, threads, `asyncio` and the GIL, pytest coverage and fixtures, interpreter-level performance, public API and typing).

Every layer lens keeps the emphasis-never-exclusion posture and the no-"ignore" rule of the base.
A layer's fan override holds every lens of the base fan of that name, in base order, followed by the layer's additions.

Rationale: layers ship as data, so lyx carries no language logic in code, and a later edit to a layer reaches every hub that picked it.
The superset rule keeps a base fan edit from silently going missing in a layer.

### D3 — `lens_set` in `lenses.yaml` picks the layer, from the base layer only

`lenses.yaml` gains a top-level scalar key `lens_set`, template default `""`, classified Guard.
`""` means no layer: the base template alone.
Any other value must name a shipped layer.

The key is read only from the module's base layer: the hub file `<Board>/_lyx/config/lenses.yaml` in a hub, the worktree file standalone.
A `lens_set` key in a hub worktree file is refused at load with an `ErrInvalid` error naming the file, saying the key belongs in the base file, and naming the way forward: remove `lens_set` from the named file, then set it in the base with `lyx config lenses --layer default --set lens_set=<name>` from the hub's prime worktree.
`configengine.Set` refuses a write that leaves the selector key in a hub worktree file before anything is written,
and `configengine.Edit` refuses such a save after the editor writes it, putting the file back to its previous bytes or removing it when the call created it, as it does for a refused local save;
both give the same refusal, which names the removal step only when the file held the key before the write.
So `lyx config lenses --set lens_set=<name>` in a hub worktree gives that refusal alone, never followed by the generic load-failure way forward.
A `lens_set` key in a local file keeps the refusal `ValidateLocal` gives every Guard key, since that check runs first; lenses, a per-worktree module, has no machine layer.
An unknown value is refused at load with an `ErrInvalid` error naming the shipped layers and the way forward: `lyx config lenses --layer default --set lens_set=<name>` from the hub's prime worktree in a hub, `lyx config lenses --set lens_set=<name>` standalone.

Rationale: every file is judged against the template the load uses, so the selector must have one source for the whole stack.
The guard: a worktree's own `lenses.yaml` can no longer pick a layer; it still overrides any lens or fan by name.

### D4 — Configengine composes the effective template; every judging path uses it

`configengine.Spec` gains an optional overlay: the name of the selector key and a map from selector value to overlay document.
lenses fills it from its embedded layers; every other module leaves it empty and behaves exactly as today.

The effective template is the base template with the picked overlay's per-name entries laid over it, each overlay name replacing or adding one entry whole.
The selector is always read from the module's base layer, whatever document is being judged: the hub file in a hub, the worktree file standalone, and the document being written when the write targets the base layer.
Configengine exports the one function that composes the effective template, and no other package composes it.

A path is classified by what it does, whatever verb runs it:

- Every path that compares a document against the template, or composes the template with the layers below a given layer, uses the effective template: configengine's load and fallback, `Set`, `Edit`, `Provenance` and below-layer composition, and every configsync strip, prune, reconcile and template-equal drop, on a base file, a worktree file or a seed alike.
- Only paths that list template keys or scaffold a whole template keep the base template, a scaffold keeping it outside the per-name maps of an overlay module, as the next bullet says.
  The names an overlay adds, such as `numerics`, live in per-name maps, where those paths keep a name the template lacks.
- A scaffold of a base file for a module whose spec carries an overlay, through `Set` or `Edit`, writes its per-name maps empty, as standalone reconcile already writes a fresh file, so no scaffold pins a base template entry over the picked layer.

`Provenance` labels a unit the overlay supplies as `template:<lens_set>`.

Rationale: reconcile's promise that a drop leaves the loaded value unchanged holds only if load and every judging path use the same template.
Under `lens_set: csharp`, an entry equal to the csharp prose is dropped and one equal to the base prose is kept, in the hub file and in a standalone worktree file alike; a hub worktree file is only pruned, and the one-time migration strip judges it against the layers below it.
A generic spec field keeps lenses a leaf package and keeps layer resolution inside configengine (`PATTERN-config-layers`, `PATTERN-leaf-packages`).

### D5 — `remove` drops lenses after resolution

`lenses.yaml` gains a top-level list key `remove`, template default `[]`, classified Guard; it stacks like any list, a higher layer replacing it whole.
`lenses.Load` applies it after the resolved document is decoded, and is the only place it applies: every lens consumer reads through `lenses.Load`, and a raw read of the resolved document sees no removal.
Each named lens leaves `Lenses`, and every occurrence leaves every fan, the hub's own fans included.
A fan that removal leaves empty leaves `Fans`, and `ResolveFan` on it refuses with an error saying `remove` emptied it, naming the lenses removed and the way forward: point the fan key at another fan or `""`, or drop a name from `remove`.
A `remove` entry naming no lens of the resolved document is skipped, so a lens a later template edit or a layer switch takes away never fails a load.
`lenses.Load` reports the skipped entries through a helper configengine exports, which logs them at Warn naming the layer file the resolved `remove` came from and each entry, once per process per file and entry list, the way `warnUnknownKeys` dedups, so a loader in a poll loop warns once.
`CheckLoad` resolves the fans that remain.

The guard: `remove` lets a hub take a review lens out of every fan.
Its bound: a Guard key, so no local override and no run setting can hold it; only a tracked base or worktree file can, a worktree file's `remove` replacing the hub's whole, and the fans it shortens still run their remaining lenses.

Rationale: the operator decided each repository can remove lenses it does not want; a removal applied in lenses keeps fan logic out of configengine.

### D6 — The operator picks the layer; existing hubs move to the neutral base until then

No code writes `lens_set` for a hub.
A hub's operator sets it once with D3's set command, `lyx config lenses --layer default --set lens_set=<name>` from the hub's prime worktree, which runs `CheckLoad` and rolls back an unknown name.
This hub and the quarry hub, both Go, resolve the neutral base from landing until the operator sets `lens_set: go` on each; the orch runs it for this hub right after landing.
This is accepted: the base lenses still review every area, and the task worktree cannot write Bolt's config.
`pattern-fit`, `code-pattern-fit` and `docs-consistency` change prose for every hub, Go hubs under `lens_set: go` included, since D1 rewords them in the base.

Rationale: a hub's base file holds only what the hub tuned (`PATTERN-sparse-hub-base`), and choosing a layer from repository files would be language logic in code.

### Added after Discussion (parent, 2026-10-22): D3 refinement: an equal lens_set in a hub worktree file is allowed

Decision: D3 refinement: a hub worktree file's lens_set is refused only when its value differs from the base layer's ("" when the base lacks the key), at load and by Set and Edit alike. An equal value changes no selection, and the sparse migration strips it as equal to the layers below.

Rationale: The template carries lens_set: "", so a full worktree lenses.yaml written from it holds the key (hubreconcile's commitPreMigrationFiles fixture, a standalone-reconciled file whose repo becomes a hub); refusing any key would make readLayers refuse the file inside the walk that strips it.

### Added after Discussion (parent, 2026-10-22): Shorten PATTERN-sparse-hub-base for headroom

Decision: The PATTERN-sparse-hub-base line drops its walk-strip clause, which its background file already states.

Rationale: The new PATTERN-template-overlay line (171 bytes) pushes PATTERN.md past patternlint's headroom line; cutting the intro saves only about 82 bytes.

### Added after Discussion (parent, 2026-10-22): Shorten PATTERN-config-layers for headroom

Decision: The PATTERN-config-layers line drops its machine-layer clause, which its background file already states.

Rationale: Same headroom need as the PATTERN-sparse-hub-base cut: the new PATTERN-template-overlay line must fit under patternlint's headroom line.

## Constraints

- `PATTERN-config-strictness`: lenses keeps `LoadOrTemplate`.
- Every new template key is classified in `ConfigSpec` (`TestModules_EveryTemplateKeyClassified`).
- `PATTERN-leaf-packages`: `internal/lenses` gains no import; the `remove` Warn is logged through configengine.
- `PATTERN-documentation-lifecycle`: the design lands in `internal/lenses/doc.go` and `internal/configengine/doc.go`.

## Auto-mode assumptions

- The selector key is named `lens_set`: `set` reads ambiguously with `--set`, and `language` would compete with a hub-language key a later entry may add to `language.yaml`.
- The rules lenses stay in the base with neutral wording, against the board's candidate list, which named them for the layers.
- `numerics` sits in the `csharp` layer and runs by default in a C# hub's `review-code`, at one extra fork per code round 1.

## Open risks

- A `remove` that empties a fan a loom or darn fan key names passes `lyx config lenses --set`; it surfaces when loom or darn validates or resolves that fan key, at its config check or at the run's next review round.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - worktree-template-equal-drop-in-hub-nonexistent (class consistency, severity MEDIUM, open in rounds 1, 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-1-review.md; fixer report: _lyx/reviews/plan/round-1-fixer-report.md.
  - lensesfixture-leaks-new-config-fields (class scope, severity BLOCKING, open in rounds 1)
  - strip-moved-units-judgment-unpinned (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- With `lens_set` absent or `""`, a load resolves the base template's lenses and fans, every one language-neutral.
- With `lens_set: go`, the five overlaid lenses resolve to today's prose verbatim, and every other lens and fan to the base.
- With `lens_set: csharp`, the five lenses resolve to the csharp prose, `numerics` is defined, and `review-code` and `full` end with `numerics`.
- A hub-file or worktree-file lens entry beats the layer's entry of the same name.
- Under a hub file with `lens_set: csharp`, `lyx config reconcile --layer default --apply` from the hub's prime worktree drops a hub-file lens entry equal to the csharp prose and keeps one equal to the base prose, and the loaded document is the same before and after.
- In a standalone worktree whose file holds `lens_set: csharp`, `lyx config reconcile --apply` drops a lens entry equal to the csharp prose and keeps one equal to the base prose, and the loaded document is the same before and after.
- An unknown `lens_set` value is refused naming the shipped layers and the way forward; a `lens_set` key in a worktree file above a hub base is refused naming the file, the removal of the key from it and `--layer default`; one in a local file is refused as a guard key.
- In a standalone worktree with no `lenses.yaml`, `lyx config lenses --set lens_set=csharp` resolves the five lenses to the csharp prose.
- `lyx config lenses --layer default --set lens_set=xyz` is rolled back with the unknown-value error.
- In a hub worktree whose `lenses.yaml` holds no `lens_set`, `lyx config lenses --set lens_set=csharp` writes nothing and is refused naming `--layer default` from the prime, with no other way forward.
- `remove: [numerics]` under `lens_set: csharp` leaves `numerics` undefined and absent from every fan.
- `remove` that empties a fan drops the fan, `CheckLoad` passes, and `ResolveFan` on that fan names `remove` and the way forward.
- A `remove` entry naming no lens loads, logged at Warn once per process.
- For every shipped layer, every fan resolves (`CheckLoad` passes), no lens text carries "ignore " phrasing, the layer holds only `lenses` and `fans` entries, and each layer fan override starts with its base fan's lenses, exactly and in base order, followed only by the layer's additions.
- `Provenance` labels an overlay-supplied lens `template:<lens_set>`.
- A module with no overlay in its spec loads, sets, edits and reconciles exactly as before.

## PATTERN changes

- `PATTERN-template-overlay` — adds: Judging a config document against its template: only against `configengine`'s effective template, the overlay picked from the base layer.
- `PATTERN-sparse-hub-base` — changes: Writing a hub's base file: only the units the hub tuned, never a template fill. — added after Discussion (parent, 2026-10-22)
- `PATTERN-config-layers` — changes: Resolving a module's config: only configengine reads, stacks and locates layer files. — added after Discussion (parent, 2026-10-22)

## Notes for the plan writer

- The base and `go` layer split is a move: lift today's prose of the five lenses into `sets/go.yaml` verbatim before rewording the base.
- `internal/lenses/config_test.go` pins seeded lens and fan counts and the no-"ignore" scan over the template; the layer checks extend those tests rather than adding parallel ones.
