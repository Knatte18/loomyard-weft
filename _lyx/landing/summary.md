# lyx config --set writes a per-round model list as a list

`lyx config <module> --set` on a key that takes one model-spec or a per-round list kept the template's string type.
`lyx config loom --set 'plan_review=["opus[medium]","sonnet[high]"]'` wrote a quoted string, which is not a valid model-spec, and the operator had to revert it in an editor.
`--set` now writes the shape the value spells, so a per-round model change needs no editor session.

## Behaviour

- A key that is itself a declared open-map path whose template value is not a mapping takes either shape.
  In loom these are `review`, `fix`, `fan_review` and each segment's `*_review` and `*_fix`.
- A value whose first non-blank character is `[` is parsed as a YAML flow list of scalars and written as a list.
  Any other value is written as a string scalar.
- A bracketed value that is not a flow list of scalars is refused with an error naming the key and the quoted form, and nothing is written.
  This covers unquoted bracketed entries such as `[opus[medium], sonnet[high]]`, nested lists, and text after the closing bracket.
- The deciding shape is the value's spelling, not the current node's, so `--set plan_review=opus[high]` replaces a list with a scalar, and `--set 'review=[]'` writes an empty list.
- The replaced node keeps its comments.
- `P.<name>` entries and mapping-valued open maps such as board's `labels` behave as before.

## Design decisions

- The change is in `internal/yamlengine`'s `SetValues` and `SetSparse`, so it reaches the full base layer and the sparse worktree and local layers alike.
  `configengine` is untouched.
- Whether a path takes the new branch is decided from the template's own value for P, read before `carryOpenMaps` runs.
  Deciding from the carried file value would let `--set labels=x` overwrite a list-shaped `labels` under a mapping-declared open map.
- No new registry field: loom already declares these keys in `loomengine.ConfigOpenMaps`.
- Model-spec grammar is not checked on write; loom's load-time validation and the local layer's `CheckValues` stay the checks.
- Risk: a hand-written value starting with `[`, meant as a literal string at such a key, is now parsed as a list.
  No current key takes such a string, and no model-spec starts with `[`.

## Docs

`lyx config --help`, `internal/configcli/doc.go`, the open-map section of `internal/yamlengine/doc.go` and `docs/overview.md` describe the new form.
The help text no longer says a list-shaped open-map value must be rewritten in an editor first.

## Tests

The `SetValues` and `SetSparse` table tests gained rows for the quoted flow list, the scalar-to-list and list-to-scalar moves, the unquoted bracketed refusal, trailing text after the bracket, and a list-shaped `labels` under a mapping-declared open map.
The empty-list and nested-list rows are on the `SetValues` table.
`lyx gate test ./...` and `lyx gate test --tags integration ./...` were not run as part of this description.

## Check by hand

- `lyx config board --set labels=x` with `labels` stored as a list in the file: it must still be refused, with the file unchanged (`internal/yamlengine/set.go`, `scalarOrListOpenMapPaths`).
- `lyx config loom --set 'plan_review=["a"] trailing'`: it must be refused, with nothing written (`internal/yamlengine/set.go`, `scalarOrListNode`).
- `internal/yamlengine/layers_test.go`: `TestSetSparse` has no empty-list or nested-list row; confirm the shared `scalarOrListNode` coverage in `TestSetValues` is enough.
