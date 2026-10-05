# Support log — board-labels-and-skills

## Interview

Autonomous session: no operator answers; every pick below is an auto-pick recorded in the question ledger.

1. Read the board entry as first written: label descriptions, CLI tree exceptions, ly-drive as a stencil, imperative skill loading.
   Explored stencils, `driverPrompt`, the ly-drive skill and its pinning tests, and drafted (not written) a design for the stencil and skill-loading parts.
2. Mid-session, the orch relayed an operator scope change: ly-drive and skill loading moved to `lyx-skills-and-stencils`; board CLI gaps were added; `types` get descriptions too.
   Re-read the board entry; dropped every draft decision for the moved parts.
3. Labels: found that `yamlengine.Reconcile` drops mapping keys absent from the template and `MissingKeys` demands template keys, so a user-keyed map cannot survive today's config machinery; lists already have a carry-whole rule.
   Burler's `lenses`/`fans` are the existing precedent of the same exposure.
   Picked a per-module open-map declaration mirroring the list rule (D3).
4. Live `board.yaml` files are lists in every worktree's weft; picked decode-only tolerance of the list shape (D2) over a hub-wide break at deploy.
5. Board CLI gaps: read `get`, `merge`, `bodyfile.go`, `MergeTasks`, `maxSlugLength`; settled D5–D8.
6. CLI tree: `cliTreeFindings` holds only the three config/selfreport entries; the `board notes` ones went with `ee1ffa4b`.
   `lyx config` bare runs an interactive picker after resolving a cwd; picked help listing and early refusal (D9).

## Rejected alternatives

- Labels as a list of single-key maps (`- board: …`): survives reconcile with no config change, but it is not the map the operator asked for and reads worse.
- A YAML tag (`!open`) in the template marking open maps: reconcile marshals the template node, so the tag would leak into users' files and into decoding.
- Treating every empty template mapping as open: does not cover `types`, whose template seeds `bug` and `enhancement`.
- Strict refusal of the list shape with a way-forward message: breaks every board call in every worktree until each file is hand-edited.
- Keeping `undecided` in the template: contradicts the labels principle.
- `lyx board get` with `task: null` semantics under `--body` (print nothing, exit 0): indistinguishable from an empty body.
- Merge issue carry behind a flag: the hub always wants provenance kept; a flag is a way to lose it.
- Hard-coding `32` in help strings: drifts from the validator.
- Keeping the `lyx config` picker on a TTY and listing otherwise: two behaviors for one invocation, and the walk tests the non-TTY one only.
- Moving the picker to `lyx config menu`: nobody asked for it; `lyx config <module>` covers editing.
- Keeping an empty `cliTreeFindings`: invites the next exception.
- Fixing burler's maps with the same declaration: not asked for; noted as risk R3.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): shape of `types`/`labels` and how config survives it → ordered maps plus a declared open-map rule (D1, D3).
- Q2 (resolved, auto-pick): what happens to existing list-shaped files → decode-only tolerance (D2).
- Q3 (resolved, auto-pick): template default labels → `labels: {}`, `undecided` dropped; `types` seeds `bug`, `enhancement` with descriptions (D1).
- Q4 (resolved, auto-pick): `lyx board labels` output shape → JSON with separate `types` and `labels` lists in file order, no `--text` (D4, A5).
- Q5 (resolved, auto-pick): `get --body` on absent target → error envelope, exit 1 (D5).
- Q6 (resolved, auto-pick): merge issue order and conditions → upserted's then removed entries' in `remove_slugs` order, deduped, unconditional (D7).
- Q7 (resolved, auto-pick): slug limit source → exported constant formatted into each `Long` (D8).
- Q8 (resolved, auto-pick): fate of the `lyx config` picker → removed; bare prints help (D9).
- Q9 (resolved, auto-pick): keep or delete the findings allowlist → delete (D11).
- Q10 (resolved, operator via orch): ly-drive stencil and skill loading → out of scope, moved to `lyx-skills-and-stencils`.
- Q11 (resolved, operator via orch): do `types` get descriptions → yes, stored like other labels.
- Q12 (resolved, auto-pick): who writes loomyard's own label descriptions → the operator in the prime's live config after landing (A3).
