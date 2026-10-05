# Board label descriptions, board CLI gaps, CLI tree fixes

Board labels now say what they cover, the board CLI no longer needs hand-built JSON for common hub work, and the CLI tree walk has no recorded exceptions left.

Closes #349

## What changes

- `types` and `labels` in `board.yaml` are maps from label to a one-line description.
  `lyx board labels` prints both lists with their descriptions, in file order.
  The `ly:board` skill tells agents to read it before choosing labels, and states the rule: a label names the lyx module an entry mainly touches, and an entry that belongs to no module carries none beyond its type.
- `lyx config board --set labels.<name>="…"` adds or rewrites one label.
- `lyx board get --body` prints an entry's body alone and verbatim, so it round-trips through `upsert --body-file`.
- `lyx board merge --body-file <path>` behaves like `upsert`'s flag: `-` reads stdin, and a payload that also has `body` is refused.
- A merge now carries the removed entries' `issues` onto the upserted entry: its own issues first, then each removed entry's in `remove_slugs` order, without duplicates.
  Merging removed entries `[3]` and `[3, 5]` into an entry holding `[1]` yields `[1, 3, 5]`.
  The carry is unconditional.
- The `--help` of every slug-taking board verb states the slug length limit, formatted from the now-exported `boardengine.MaxSlugLength`.
- Bare `lyx config` prints help naming `reconcile` and every module, exit 0, without resolving a cwd.
  `lyx config bogus` and `lyx selfreport bogus` refuse with the shared `unknown subcommand` envelope, exit 1.
  The interactive picker in `internal/configcli/menu.go` is removed; `lyx config <module>` is still the way to edit one.
- `cliTreeFindings` and the known-finding path in `cmd/lyx/clitree_test.go` are deleted, so every command must meet the CLI invariant outright.
  The `lyx board notes` exceptions were already gone.

## Design decisions

- **Open maps in config.**
  A module declares the key paths of its open maps next to its template (board: `types`, `labels`), and the declaration reaches `configengine` load and `Set` and `configsync` reconcile through `configreg.Module.OpenMaps`.
  For a declared path, `yamlengine` treats the value as it treats lists: reconcile carries it whole, `FillMissing` fills it only when absent, and `MissingKeys` is satisfied by the key's presence.
  `Reconcile`, `MissingKeys`, `FillMissing` and `SetValues` take a trailing `openMaps ...string`.
  Undeclared mappings keep exact-key behavior, so a typo in an ordinary key is still reported.
  Without this, `reconcile --apply` would delete every repo-specific label.
- **Consequence of the above, by design.**
  Nothing checks the shape under an open map except `boardengine`'s decode, and `--set labels.<typo>=…` creates a label rather than refusing.
  `Set` under an open map writes scalars only; removing a label is an editor edit.
- **Legacy list shape still loads.**
  `boardengine` decodes a `types` or `labels` sequence of names as labels with empty descriptions, in file order, as `[]Label` rather than a Go map, because the README renderer orders Notes by the configured type order.
  Nothing writes the list shape, and `--set labels.<name>` on a list-shaped value refuses with a message to rewrite it as a map.
  Every live worktree's `board.yaml` is still a list, and a strict refusal would break every board call until each was rewritten.
- The template drops `undecided` and seeds `types` with `bug` and `enhancement`; `labels` is an empty map.
- `lyx board labels` has no text form; agents read JSON.
- New `doc.go` files for `yamlengine`, `configengine` and `configcli`; `boardengine` and `boardcli` docs, `SKILL.md` and `tools/sandbox/SANDBOX-CORE-SUITE.md` are updated, which is where the sandbox suite picks up `board labels`.
  No `PATTERN.md` entry: open maps are a config-template semantic owned by `yamlengine`.

## To check by hand

- No live `board.yaml` is edited here, so `lyx board labels` shows empty descriptions until the operator rewrites each file as maps, starting with the prime's, using `lyx config board --set labels.<name>=…` or the editor.
- An older `lyx` binary running `lyx config reconcile --apply` against a map-shaped `board.yaml` treats the repo labels as removed keys and deletes them.
  Watch for a stale `.dev-bin` or a second machine after deploy.
- The plan-level verify and the full `go test ./...` plus sandbox suite had not run when the run record was written; run them.
- Two batches edited a test file through a shell heredoc instead of `Edit`; the resulting diffs are correct, but give `internal/boardcli` and `internal/boardengine` tests a read.
