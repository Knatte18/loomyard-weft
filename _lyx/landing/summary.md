# Lenses resolve from a neutral base plus a language layer

A hub's lenses now resolve from a language-neutral base template plus an embedded language layer the operator picks, so a C# hub no longer starts with lenses worded for this Go repository.
A hub can still add and override lenses and fans by name, and can now remove a lens it does not want.

## What changes

- `internal/lenses/template.yaml` keeps every lens and fan name and drops all language wording.
  `error-handling`, `test-gaps`, `performance`, `api-design` and `concurrency` are reworded neutrally.
  `pattern-fit`, `code-pattern-fit` and `docs-consistency` now read the repository's own rules where it keeps them: `PATTERN.md` and `pattern/` when present, otherwise its `CLAUDE.md` files.
  This rewording reaches every hub, Go hubs under `lens_set: go` included.
- `internal/lenses/sets/` holds one layer per language: `go`, `csharp` and `python`.
  A layer holds only `lenses` and `fans` entries, each replacing the base entry of that name whole or adding a name.
  `go` carries the previous wording of the five reworded lenses verbatim.
  `csharp` carries C# wording and adds a `numerics` lens (floating-point comparison, overflow, units, out-of-range physical values); its `review-code` and `full` fans end with `numerics`.
  Its `error-handling` and `numerics` prose defer to the repository's own rules on guards and exception handling.
  Each layer fan override holds the base fan's lenses in base order, then the layer's additions.
- `lenses.yaml` gains `lens_set` (default `""`, the base alone) and `remove` (default `[]`), both Guard keys.
- `remove` is applied in `lenses.Load` after resolution.
  A removed lens leaves `Lenses` and every fan; a fan left empty leaves `Fans`, and `ResolveFan` on it names `remove` and the way forward.
  An entry naming no lens is skipped and logged at Warn once per process per file and entry list.

## Design decisions

- `configengine.Spec` gains an optional overlay (selector key plus a map from value to overlay document); lenses fills it, other modules leave it empty and behave as before.
  `configengine` exports the one function that composes the effective template, and every path that judges a document against the template uses it: load, `Set`, `Edit`, `Provenance`, and every `configsync` strip, prune, reconcile and template-equal drop.
  Under `lens_set: csharp`, reconcile drops an entry equal to the csharp prose and keeps one equal to the base prose, and the loaded value is unchanged.
  Paths that list template keys or scaffold a whole template keep the base template.
  A base-file scaffold for an overlay module writes its per-name maps empty, so no scaffold pins a base entry over the picked layer.
  `Provenance` labels an overlay unit `template:<lens_set>`.
  `PATTERN-template-overlay` records the invariant.
- `lens_set` is read only from the module's base layer: the hub file in a hub, the worktree file standalone.
  A hub worktree file holding a differing `lens_set` is refused at load with an `ErrInvalid` naming the file and the way forward (`lyx config lenses --layer default --set lens_set=<name>` from the prime worktree).
  An equal value is allowed, since a full worktree file written from the template carries `lens_set: ""` and the migration strip drops it.
  `Set` refuses before writing; `Edit` refuses after the editor writes and restores the file's previous bytes, or removes the file when the call created it.
  An unknown value is refused at load naming the shipped layers.
- No code detects a hub's language or writes `lens_set` for a hub.
  This hub and the quarry hub resolve the neutral base until the operator runs `lyx config lenses --layer default --set lens_set=go` from each hub's prime worktree.
- No reworded lens gets a `formerTemplateValues` entry in `configsync`: the `go` layer carries their old prose, and stripping a hub's pinned copy would change its loaded value under another layer.
- `PATTERN-sparse-hub-base` and `PATTERN-config-layers` each lose a clause their background file already states, to keep `PATTERN.md` under `patternlint`'s headroom.
- `internal/lenses` gains no import; the `remove` Warn is logged through a helper `configengine` exports.
- Docs are updated in `internal/lenses/doc.go`, `internal/configengine/doc.go`, `internal/configcli/doc.go`, `internal/configsync`'s package doc and `docs/overview.md`.

## Check by hand

- Set `lens_set: go` on this hub right after landing with `lyx config lenses --layer default --set lens_set=go` from the prime worktree; until then its lenses resolve the neutral base.
- Not run against a real hub: the `csharp` and `python` layer wording, and `numerics` running by default in a C# hub's `review-code` at one extra review fork per code round 1.
- A `remove` that empties a fan named by a loom or darn fan key passes `lyx config lenses --set`; it surfaces when loom or darn validates or resolves that fan key.
- Finding `lensesfixture-leaks-new-config-fields` (scope, blocking): check that the lenses test fixture does not leak the new `lens_set` and `remove` fields into other modules' fixtures or tests.
- Finding `strip-moved-units-judgment-unpinned` (design): check the `configsync` strip of moved units, which judges against the effective template; no test pins that judgment for every moved unit.
- Finding `worktree-template-equal-drop-in-hub-nonexistent` (consistency): check that a hub worktree file is only pruned, with its template-equal drop judged against the layers below it, and that no standalone-only template-equal drop applies in a hub.
