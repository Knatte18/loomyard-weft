# Board label descriptions, board CLI gaps, CLI tree exceptions

Board labels now say what they cover, the board CLI no longer needs hand-built JSON for the hub's merge workflow, and the CLI tree walk in `cmd/lyx/clitree_test.go` has no recorded exceptions left.

Closes #349

## What changes

- `types` and `labels` in `board.yaml` are ordered maps from label to a one-line description.
  `lyx board labels` prints both in file order, so an agent can read every valid label and what it covers before choosing.
  The `ly:board` skill states the labels principle: a label names the lyx module an entry mainly touches, and an entry that belongs to no module carries none beyond its type.
- `lyx config board --set labels.<name>="description"` adds or rewrites one label.
- `lyx board get --body` prints an entry's body alone and verbatim, so it round-trips through `upsert --body-file`.
- `lyx board merge --body-file` works like `upsert`'s, including `-` for stdin.
- A merge carries the removed entries' `issues` onto the upserted entry, deduplicated, first occurrence kept.
- The `Long` help of every slug-taking board verb states the slug length limit, formatted from the exported `boardengine.MaxSlugLength`.
- Bare `lyx config` lists its subcommands and modules, and `lyx config bogus` and `lyx selfreport bogus` refuse with an `unknown subcommand` error.
  `cliTreeFindings` is deleted.
- The module picker returns as `lyx config menu`.
- The editor fallback tries `$VISUAL` and `$EDITOR`, then `code --wait`, then `nano`, then `vi`, with notepad kept on Windows.

## Design decisions

- **Open maps.** A config module declares the key paths of its open maps next to its template (board: `types` and `labels`).
  `yamlengine` then treats them like lists: reconcile carries the existing value whole, `FillMissing` fills only when the key is absent, and `MissingKeys` is satisfied by presence.
  `Reconcile`, `MissingKeys`, `FillMissing` and `SetValues` take a trailing `openMaps ...string`, and `configreg.Module` gains `OpenMaps`.
  Undeclared mappings keep exact-key behavior.
  Tradeoff: `--set labels.<name>=…` with a mistyped name creates a label, and `boardengine`'s decode is the only shape check on those keys.
  `--set` writes scalars only and refuses a legacy list-shaped value; removing a label stays an editor edit.
- **Legacy list shape.** A `types` or `labels` sequence of names still loads, with empty descriptions, and nothing writes it.
  Every live worktree's `board.yaml` is a list today, and a strict refusal would break every board call until each is rewritten.
- **Template.** `labels` is empty by default and `undecided` is removed; `types` keeps `bug` and `enhancement`.
- **Picker.** The bare-group invariant requires bare `lyx config` to list, so the picker moved behind `lyx config menu` instead of being deleted.
  `lyx config` listing and refusal resolve no cwd.
- **Issue carry on merge** is unconditional and adds no cross-entry uniqueness check.

## For the reviewer

- No live `_lyx/config/` file is edited.
  Existing `board.yaml` files keep list-shaped labels with empty descriptions until the operator rewrites them as maps, for example with `lyx config board --set labels.<name>=…`.
- An older `lyx` binary running `lyx config reconcile --apply` against a map-shaped `board.yaml` treats the repo labels as removed keys.
  Check for a stale `.dev-bin` or second machine after deploy.
- Not run by hand: `lyx config menu` interactively, and the editor fallback on a machine with none of `code`, `nano` or `vi` present.
