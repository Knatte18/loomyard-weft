---
format: 5
approved: false
language: go
---

# Plan: board labels and skills

Board labels gain descriptions: `types` and `labels` in `board.yaml` become ordered maps from label to description, with the list shape still loading, and `lyx board labels` prints them.
The config machinery learns open maps, declared per module and carried whole through load, reconcile and `--set`, so repo-specific labels survive.
The board CLI closes the hub's gaps (`get --body`, `merge --body-file`, issue carry-over on merge, the slug limit in help), and `ly:board` documents them with the labels principle.
`lyx config` and `lyx selfreport` meet the CLI tree invariant, the `lyx config` picker goes, and `cliTreeFindings` is deleted.

## Card Index

1 — yamlengine-open-maps — Reconcile, FillMissing, MissingKeys and SetValues carry declared open maps whole
2 — configengine-open-maps — Load, LoadOrTemplate and Set pass a module's open maps to yamlengine
3 — board-label-maps — board.yaml types and labels decode as ordered maps, lists still load
4 — configreg-open-maps — configreg declares board's open maps and reconcile carries them
5 — config-set-open-maps — lyx config --set labels.<name>=… adds or rewrites one label
6 — board-labels-verb — lyx board labels prints both maps in file order; skill states the labels principle
7 — board-get-body — lyx board get --body prints the body verbatim
8 — board-merge-body-file — lyx board merge takes --body-file
9 — merge-issue-carry — a merge carries the removed entries' issues
10 — export-slug-limit — rename maxSlugLength to MaxSlugLength
11 — slug-limit-help — every slug-taking board verb's help states the limit from the constant
12 — config-bare-and-bogus — bare lyx config lists, an unknown argument refuses, the picker goes
13 — selfreport-bogus — lyx selfreport refuses an unknown subcommand
14 — clitree-findings-gone — delete cliTreeFindings and the known-finding path

## Shared Decisions

- Open maps are passed as a trailing variadic `openMaps ...string` on yamlengine's and configengine's entry points, so no existing caller changes and `PATTERN-config-strictness`'s pinned call sites stay put.
- A module declares its open maps beside its template (`boardengine.ConfigOpenMaps`); `configreg.Module.OpenMaps` carries the declaration to `configsync` and `configcli`, and `boardengine.LoadConfig` passes it to `configengine.Load` itself.
- `boardengine.Config` holds labels as `[]Label` in file order; `Vocabulary` and `Outputs` keep their `[]string` names, so validation and rendering are untouched.
- Docs land with the card that changes the behaviour: new `doc.go` files for `yamlengine`, `configengine`, `configcli` and `boardcli`, and `SKILL.md` edits in the board verb cards.
- No live `_lyx/config/` file is edited.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```bash
go build ./...
go test ./...
go test -tags integration ./internal/yamlengine/... ./internal/configengine/... ./internal/configreg/... ./internal/configsync/... ./internal/configcli/... ./internal/boardengine/... ./internal/boardcli/... ./internal/selfreportcli/... ./cmd/lyx/...
go vet -tags smoke ./...
```
