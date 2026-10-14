# config --set writes a per-round model list as a list

`lyx config loom --set 'plan_review=["opus[medium]","sonnet[high]"]'` wrote `plan_review` as a quoted string, not a list.
Loom's per-round model keys (`review`, `fix`, `fan_review`, `discussion_advisors` and each segment's `*_review` and `*_fix`) take one model-spec or a per-round list.
`--set` chose the written shape from the key's current node, so a key holding a scalar always got a string, and a key holding a list refused a scalar.

## What changes

- `internal/yamlengine`: `SetValues` and `SetSparse` treat a key that is itself a declared open-map path, and whose current value is a scalar or a list rather than a mapping, as taking either shape.
  A value whose first non-blank character is `[` is parsed as a YAML flow list of scalars and written as a list; any other value is written as a string.
  The replaced node's comments are kept.
  A bracketed value that does not parse, or parses to anything but a list of scalars, is refused with an error naming the key and the quoted-entry form, and nothing is written.
- Loom already declares these keys through `loomengine.ConfigOpenMaps`, so no registry or spec field is added.
  Mapping-valued open maps such as board's `labels` are unaffected: `P.<name>` keeps its entry behaviour and `P` itself stays outside the set path.
- `internal/configcli/doc.go`, `internal/yamlengine/doc.go` and the config entry in `docs/overview.md` describe the new form, and `lyx config --help` gives one example.
  To keep the help under its length cap, the open-map caveats (removing an entry, a list-shaped map) left the help; the package doc already states them.

## Design decisions

- The shape follows the value's spelling, not the key's current shape, so `--set` can move a key from a scalar to a list and back.
  No model-spec starts with `[`, so the rule never misreads a scalar.
- The unquoted form `[opus[medium], sonnet[high]]` is refused, not accepted: YAML forbids `[` and `]` in an unquoted flow-list entry, and a lyx-specific list grammar for a handful of keys was not worth a second parser.
  The refusal names the quoted form as the way forward.
- `--set` does not check model-spec grammar; loom's load-time validation and the local layer's `CheckValues` stay the checks.

## Tests

`TestSetValues`, `TestSetValues_Refusals` and `TestSetSparse` gain rows for a quoted flow list, a list key taking a scalar, the empty list, the unquoted bracketed refusal and a nested-list refusal, on both the full and the sparse write paths.
