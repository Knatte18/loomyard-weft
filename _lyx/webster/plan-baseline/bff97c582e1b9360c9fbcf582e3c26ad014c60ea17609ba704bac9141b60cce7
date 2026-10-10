---
format: 5
approved: true
language: go
---

# Plan: cli-surface

Give the orch every `lyx` command it may use without reading `--help` first, make every command's help read the same way, and carry an issue's comments onto the board.
A `help` module adds `lyx help index`, a generated one-line-per-command index of the running binary filtered by an `audience` annotation every invocable command carries; the orch's role file carries that index on every launch and reload, read from the `lyx` the orch pane runs; an operator skill loads it on demand; one test-enforced form governs `Use`, `Short` and `Long`, the board payload commands name their keys in `Use`, `set-deps` addresses by `id`, and `lyx board intake` carries issue comments.

## Card Index

1 — audience-and-index-renderer — declare the `audience` annotation and build the index renderer in clihelp
2 — help-module — add `internal/helpcli` with `lyx help [<command>...]` and `lyx help index --audience`, mounted as the root's help command
3 — shed-verbs-audience — give the shared shed verb bodies an audience per mount and bring the batten and shed mounts to the form
4 — small-modules-form — bring burler, config, gate, ide, quarry, selfreport, shuttle and stencil to the form
5 — reed-orch-form — bring reed and orch to the form
6 — fabric-form — bring fabric to the form, trimming its long pages into a package doc
7 — loom-form — bring loom to the form, trimming its long pages into a package doc
8 — webster-form — bring webster to the form, trimming its long pages into a package doc
9 — board-payload-use — name every board payload's keys in `Use`, state the grammar once, couple `Use` to the parsers, and give `set-deps` id addressing
10 — issue-comments-fetch — read an issue's comment count and fetch its comments in selfreportengine
11 — intake-comments-body — append issue comments to the imported or folded body in boardengine
12 — intake-carries-comments — show the count in `intake list` and carry filtered comments through `intake import`
13 — strand-lyx-bin — record on each strand the `lyx` path its pane prelude exported
14 — role-file-command-index — render the operator index into the orch role file at every launch and reload site
15 — operator-cli-skill — add the `ly:cli` skill that loads the index into an operator's session
16 — cli-form-guard — pin the audience, `Use`, `Short` and `Long` form with one tree walk and record it in PATTERN

## Shared Decisions

### Decision: rewrite-before-guard

- **Decision:** the per-package rewrite cards (3 to 9) land before the `cmd/lyx` form test (card 16), each green under its own package tests.
- **Rationale:** the record notes the form test fails until every command is rewritten; sequencing the rewrites first keeps every intermediate commit green and each package's change reviewable on its own, while the final guard pins the whole tree at once.
- **Applies to:** cards 3 to 9, 16

### Decision: no-newly-required-flag

- **Decision:** a flag leaves `Use` wherever cobra does not already mark it required (`goto --to`, `webster reset --to`, the fabric and gate option flags); no command newly calls `MarkFlagRequired`.
- **Rationale:** D6 admits a flag in `Use` only when the command marks it required, and cobra's own required-flag error names no way forward, which `PATTERN-refusal-way-forward` demands of a bound module's refusal; each of these verbs already refuses a missing flag with its own message.
- **Applies to:** cards 3, 4, 6, 7, 8

### Decision: audience-per-mount-through-verb-text

- **Decision:** `shedverbs.VerbText` gains an `Audience` field and each verb body writes it as the annotation, so a body mounted under `loom`, `shed` and `batten` takes its value at each mount.
- **Rationale:** D3 wants the value set where the command is built and kept nowhere else; the text table each mount already owns is that place.
- **Applies to:** cards 3, 7

### Decision: index-note-annotation

- **Decision:** the renderer appends the value of a generic `IndexNoteAnnotation` to a group's line, and boardcli sets the payload legend there from the same constant its group `Long` ends with.
- **Rationale:** D2 wants the `board` line to end with the legend and D7 wants the grammar stated once; a generic annotation keeps clihelp free of board knowledge.
- **Applies to:** cards 1, 9

### Decision: upsert-issues-key

- **Decision:** `upsert`'s `Use` token lists `issues?` beside the keys D7 enumerates.
- **Rationale:** the store accepts `issues` from every upsert path and `merge` documents carrying issues, so D7's rule that each `Use` token equals its parser's declaration, which the coupling test enforces, outranks the record's illustrative key list.
- **Applies to:** card 9

### Decision: help-mounted-through-set-help-command

- **Decision:** `newRoot` installs the module with `SetHelpCommand(helpcli.Command())` followed by `InitDefaultHelpCmd()`, and the registration oracle reads `SetHelpCommand` arguments like `AddCommand`'s.
- **Rationale:** cobra adds a replaced help command only at execution, while the tree tests walk `newRoot()` before any execution; the oracle extension is the smaller of the two routes the record allows.
- **Applies to:** card 2

### Decision: watcher-source-seam

- **Decision:** the watcher's index source is built in orchcli from reed's strand status and an injectable runner; the runner alone spawns `<path> help index`.
- **Rationale:** every new test is tier 1, so the path selection and the missing-path refusals are tested through a fake runner, and the one spawning function is thin and logged.
- **Applies to:** card 14

### Decision: added-watcher-decisions-yield-to-d4

- **Decision:** both decisions the record adds after Discussion about the watcher yield to D4, Scope and the acceptance criteria, by one rule: where an added decision contradicts those sections, the plan follows the sections.
  "Watcher index binary" is read as the watcher rendering from the path recorded on the orch strand, which card 13 records at every launch and relaunch and card 14's source runs; the watcher's own executable is never the index source.
  "Missing-binary bound at auto-reload" yields to D4's Bound, Scope Out's one carve-out and the acceptance criterion on the watcher's render: `startAutoReload` turns a role-file render failure, the index source's included, into idle with the reason, like `startClearing` and `reloadAfterCompaction`, and never a failed tick.
- **Rationale:** each added decision contradicts several sections of the record that agree with each other.
  "Watcher index binary" declines launch-path plumbing that Scope "In", D4, the `PATTERN-pane-binary-resolution` constraint and the acceptance criteria all require, and it points at an Open risks entry that does not exist.
  "Missing-binary bound at auto-reload" rests on Scope keeping the reload steps out, while Scope Out names the `startAutoReload` change as its one exception, and D4's Bound and the acceptance criterion require idle at all three sites.
  One rule for both keeps the plan coherent, and the record's Open risks entry on a strand recorded before the field existed ("its watcher's renders go idle with the reason") then holds at every render site.
  With the path recorded on every launch, the `StartSpawnWatcher` and reed-resume mismatch the first added decision accepts does not arise, so no open risk carries it.
- **Applies to:** cards 13, 14

### Decision: classification-lives-in-the-annotation

- **Decision:** each rewrite card names the audiences it sets and the implementer confirms each once against who calls the verb, starting from the board entry's Part 1 table and the record's named examples (`reed switch`, the webster batch verbs, `burler await-review` and `fabric merge-stage` as role or internal; every `status` verb, `webster reset` and `shuttle state` as operator).
- **Rationale:** D3 keeps the classification in the annotation itself; the plan carries the rule and the starting values, not a second registry.
- **Applies to:** cards 2 to 9

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
