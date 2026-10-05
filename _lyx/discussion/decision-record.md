# Decision record — board-labels-and-skills

## Goal

Board labels say what they cover, the board CLI does what the hub needs without hand-built JSON, and the CLI tree walk in `cmd/lyx/clitree_test.go` has no recorded exceptions left.

## Scope

In:

- `types` and `labels` in `board.yaml` become maps from label to a one-line description, with a `lyx board labels` verb that prints them.
- The config machinery carries a module-declared open map through load, reconcile and `--set` without dropping or forcing keys.
- Board CLI gaps: `lyx board get --body`, `lyx board merge --body-file`, issue carry-over on merge, the slug length limit in help.
- The `ly:board` skill documents all of the above and the labels principle.
- The CLI tree exceptions still recorded: `lyx config` (bare), `lyx config bogus`, `lyx selfreport bogus`.

Out:

- ly-drive as a stencil and imperative skill loading: moved to task `lyx-skills-and-stencils` (operator scope change, 2026-10-05).
- The `lyx board notes` exceptions: already gone, removed with `notes` itself in `ee1ffa4b`.
- Writing the description text for loomyard's own labels into any live `board.yaml`: live config is per-worktree weft, and a task's config changes never reach the parent (board note `config-reaches-parent`).
- Burler's `lenses` and `fans`, which are maps with the same reconcile exposure: not asked for here.

## Decisions

### D1 — `types` and `labels` are ordered maps from label to description

Decision: both keys in `board.yaml` hold a YAML mapping, label to one-line description, in block style.
They are stored alike; `types` keeps its meaning (a note carries exactly one, a task one or more) and differs only in how it is rendered.
`boardengine` decodes each into an ordered list of (name, description) pairs in file order, never a Go `map`, because the README renderer sorts Notes sections by the configured type order.
Validation checks names only, as today; a description may be empty.
Template defaults: `types` holds `bug` and `enhancement` with generic descriptions; `labels` is an empty map, and `undecided` leaves the template.
Rationale: the operator asked for a map for both; ordered decoding keeps the renderer's type order; `undecided` contradicts the labels principle (an entry that belongs to no module carries no label), and the template is a generic default for any repo, so it seeds no module labels.

### D2 — The legacy list shape still loads, read-only

Decision: a `types` or `labels` value that is a YAML sequence of names decodes as those labels with empty descriptions.
Nothing writes the list shape: the template is a map, and reconcile carries an existing value unchanged (D3).
Bound: this is decode-only tolerance in `boardengine`; it accepts nothing a map could not express, and `lyx board labels` reports such labels with an empty description.
Rationale: every live worktree's `board.yaml` holds lists today, each in its own weft, and the board is read constantly by the hub; a strict refusal would break every board call hub-wide from the deploy until each file is rewritten by hand.

### D3 — Open maps in config: declared per module, carried whole like lists

Decision: a config module declares the key paths of its open maps (board: `types` and `labels`), next to its template, and the declaration reaches every consumer of that template: `configengine` load (`Load`, `LoadOrTemplate`), `configsync` reconcile and `configengine.Set`.
For an open-map path P, `yamlengine` applies the rule it already applies to lists:

- Reconcile carries the existing value at P whole: keys under P are neither added nor removed, and the template's value is used only when P is absent.
- `FillMissing` does not descend into P; it fills P whole from the template only when P is absent.
- `MissingKeys` is satisfied by the presence of key P, whatever its value.
- `Set` accepts any key `P.<name>`, so `lyx config board --set labels.quarry="glyphs and the quarry index"` adds or rewrites one label.

Bound: the rule applies only to the declared paths; every undeclared mapping keeps today's exact-key behavior, so a typo in an ordinary key is still reported.
`Set` under P writes scalar values only; removing a label stays an editor edit.
Rationale: without it, reconcile reports every repo-specific label as removed and `--apply` deletes it, and load re-inserts or demands the template's default labels, so a map of user-chosen keys cannot survive the existing machinery.

### D4 — `lyx board labels`

Decision: a new board verb, taking no payload, prints `{"ok":true,"types":[{"label":…,"description":…}],"labels":[{"label":…,"description":…}]}`, each list in `board.yaml` order.
It reads `board.yaml` only, through the board CLI's existing config load.
Rationale: one call gives an agent every valid label and what it covers; separate keys carry the "rendered differently" part without a second storage shape.

### D5 — `lyx board get --body`

Decision: `get` gains a `--body` flag that prints the entry's body alone, verbatim, with no envelope and nothing appended, exit 0.
An empty body prints nothing and exits 0.
A target that does not exist is an error envelope, exit 1: with `--body` there is no `task: null` to print.
Without the flag, `get` is unchanged.
Rationale: the bytes `get --body` prints round-trip through `upsert --body-file` or `merge --body-file` unchanged, so a caller edits a body as a file.

### D6 — `lyx board merge --body-file`

Decision: `merge` takes `--body-file <path>` with `upsert`'s semantics, through the same helper in `internal/boardcli/bodyfile.go`: the file's content becomes `upsert.body`; `-` reads stdin; a payload whose `upsert` already has `body` is refused, as is stdin feeding both payload and body.
Rationale: parity with `upsert`; the hub hand-built JSON to get a multi-line body into a merge.

### D7 — A merge carries the removed entries' issues

Decision: `MergeTasks` sets the upserted entry's `issues` to its own resulting `issues` (after the patch is applied) followed by each removed entry's `issues`, in `remove_slugs` order, dropping duplicates and keeping first occurrence.
It applies whether or not the payload names `issues`.
Bound: a merge can only add issue numbers that a removed entry recorded; it never drops one the payload or the existing entry holds.
A `remove_slugs` name that matches nothing contributes nothing, as today.
Rationale: GitHub provenance must survive folding notes into a task; intake's own uniqueness rule then sees each issue recorded by exactly one entry.

### D8 — Slug length limit in help

Decision: `boardengine` exports the slug length limit (today's `maxSlugLength`), and the `Long` help of every board verb that takes a slug states it, formatted from that constant: `get`, `upsert`, `upsert-batch`, `set-status`, `remove`, `merge`, `promote`, `set-deps`, `intake import`.
Rationale: the hub learned the limit only from a refused import; deriving the text from the constant keeps help and validation from drifting.

### D9 — `lyx config` bare lists, bogus refuses, before any cwd resolution

Decision: bare `lyx config` prints the command's help, naming `reconcile` and every module in `configreg`, and exits 0 without resolving a cwd.
The interactive numbered picker in `internal/configcli/menu.go` is removed.
`lyx config <arg>` where `<arg>` is neither a subcommand nor a `configreg` module refuses before cwd resolution with an error envelope, exit 1, whose message contains `unknown subcommand`, names the argument and says `run "lyx config" to list modules and verbs`.
`lyx config <module>` keeps today's behavior.
Rationale: the CLI invariant in `PATTERN-cli-cobra` requires a bare group to list and an unknown subcommand to refuse; the picker duplicated what the listing now shows, and `lyx config <module>` remains the way to edit one.

### D10 — `lyx selfreport bogus` refuses

Decision: the `selfreport` group refuses an unknown subcommand with the shared unknown-subcommand error envelope, exit 1; bare `lyx selfreport` still lists `create`, exit 0.
Rationale: the CLI invariant; the other groups already behave this way.

### D11 — The findings allowlist is deleted

Decision: with no exception left, `cliTreeFindings` and the known-finding path in `cmd/lyx/clitree_test.go` are removed, so every command must meet the invariant outright.
Rationale: an empty allowlist kept around invites the next exception; a regression gets fixed, not recorded.

### D12 — `ly:board` documents the label rules and the new verbs

Decision: `plugins/ly/skills/board/SKILL.md` says: read `lyx board labels` before choosing labels; labels name the lyx module an entry mainly touches, and an entry that belongs to no module carries none beyond its type label; `types` and `labels` are maps edited with `lyx config board`.
It also covers `get --body`, `merge --body-file`, issue carry-over on merge, and the slug length limit.
`ly:board` stays a skill; `plugins/ly/skills/INDEX.md` and the plugin description keep listing what exists.
Rationale: the skill is where agents learn the board; the labels principle is the operator's rule for choosing.

### D13 — Docs in the same commit

Decision: the change updates `internal/boardengine` and `internal/boardcli` `doc.go` (labels shape, new verb and flags, issue carry), `internal/yamlengine` and `internal/configengine` `doc.go` (open maps), `internal/configcli` `doc.go` (bare listing, picker gone), and `docs/overview.md` where it lists board or config behavior.
No `PATTERN.md` entry: open maps are a config-template semantic owned by `yamlengine`, not a cross-package invariant.
Rationale: CLAUDE.md's docs-land-in-the-same-commit rule; `PATTERN-documentation-lifecycle`.

## Constraints

- `PATTERN-cli-cobra`: the new verb has a non-empty `Short`, JSON errors via `internal/output`, `clihelp.ShouldAbort` first in `RunE`; the CLI tree walk covers it.
- `PATTERN-config-strictness`: `boardengine.LoadConfig` stays on strict `configengine.Load`; open maps change what counts as present, not the strict/degrading choice.
- `PATTERN-cliwire-sole-wiring` and `PATTERN-cwd-resolution`: D9's listing and refusal resolve no cwd; nothing new calls `os.Getwd`.
- `PATTERN-sandbox-coverage`: the new `board labels` verb is exercised by the sandbox suite or excluded with a reason.
- `PATTERN-test-tier-purity`: yamlengine, boardengine decode and CLI-tree tests stay untagged and spawn nothing; any test needing git is tagged.
- `PATTERN-markdown-link-integrity` and semantic line breaks in every edited `.md`.
- `PATTERN-documentation-lifecycle`: the built design lives in the packages' `doc.go`.
- `PATTERN-github-auth`: untouched; issue carry-over is board-local and calls no GitHub API.
- `PATTERN-lyxdirs-single-declarer`: no new `_lyx`/`.lyx` literal in production code.
- No edit to any live `_lyx/config/` file in this task.

## Auto-mode assumptions

- A1: the interactive `lyx config` picker can go; nobody depends on it beyond what `lyx config <module>` does.
- A2: the template drops `undecided`; the hub's live `board.yaml` keeps whatever it has until the operator rewrites it.
- A3: descriptions for loomyard's own labels are written by the operator into the prime's `board.yaml` after landing, with `lyx config board --set labels.<name>=…` or the editor.
- A4: legacy list tolerance (D2) stays; it is not scheduled for removal.
- A5: `lyx board labels` has no `--text` form; agents read JSON.
- A6: merge's issue carry is unconditional; no flag turns it off.

## Open risks

- R1: until the operator rewrites each worktree's `board.yaml` as maps, `lyx board labels` shows empty descriptions there; the label choice the feature exists for only improves once the prime's file carries them.
- R2: `lyx config reconcile --apply` run by an older `lyx` binary after a file is rewritten as maps would treat the repo labels as removed keys; the deploy moves every machine's binary at once, so the window is a stale `.dev-bin` or a second machine.
- R3: burler's `lenses` and `fans` keep today's exposure (repo-local lenses read as removed by reconcile); D3's declaration would fix them with one line each, outside this task.
- R4: the open-map declaration has to reach `configengine.Load` from `boardengine` and `configsync` from `configreg` without an import cycle between `configreg` and the engines; the Plan settles the exact path.

## Acceptance criteria

- `board.yaml` with `types` and `labels` as maps loads; validation accepts a label that is a key of either map and refuses any other.
- `board.yaml` with both as lists still loads, labels with empty descriptions.
- The README renderer's Notes section order follows the `types` map's file order.
- `yamlengine`: for a declared open map, Reconcile keeps every existing key and adds none from the template; a missing open-map key is filled whole; `MissingKeys` reports nothing under a present open map; an undeclared mapping still reports added and removed keys exactly as today.
- `lyx config board --set labels.x="desc"` adds the label to a map-shaped file; `--set` on an undeclared nonexistent key still refuses.
- `lyx board labels` prints `types` and `labels` in file order with descriptions.
- `lyx board get --body` prints a multi-line body byte-for-byte; an empty body prints nothing, exit 0; an absent target exits 1 with an error envelope.
- The bytes from `get --body`, fed to `upsert --body-file`, leave the body unchanged.
- `lyx board merge --body-file f` sets the body; `-` reads stdin; `body` in both payload and flag is refused.
- A merge removing entries with issues `[3]` and `[3, 5]` into an upsert holding `[1]` yields `[1, 3, 5]`.
- Every slug-taking board verb's `--help` text contains the limit formatted from the exported constant.
- From a non-git cwd: bare `lyx config` exits 0, prints no error envelope and names `reconcile`; `lyx config bogus` exits 1 with an envelope containing `unknown subcommand`; `lyx selfreport bogus` exits 1 the same way.
- `cmd/lyx/clitree_test.go` has no `cliTreeFindings` and passes over the whole tree.
- `go test ./...` passes; the sandbox suite covers `board labels`.

## Notes for the plan writer

- Board config decode: `internal/boardengine/config.go` (`Config`, `Vocabulary`, `Outputs`); template `internal/boardengine/template.yaml`.
- Merge: `Store.MergeTasks` in `internal/boardengine/store.go`; issue validation in `validateWrite`.
- Body-file helper: `internal/boardcli/bodyfile.go`; verbs in `internal/boardcli/cli.go` and `intake.go`.
- List-carry precedent for D3: `sequenceBasePath`, `applyExistingSequences` and `MissingKeys` in `internal/yamlengine/reconcile.go`; `configsync.ReconcileAll` iterates `configreg.Modules()`.
- `lyx config` routing: `Command`, `runConfig` and `dispatch` in `internal/configcli/configcli.go`, picker in `menu.go`.
