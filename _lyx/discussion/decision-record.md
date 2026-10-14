# Decision record — `lyx config --set` writes a per-round model list as a string

## Goal

`lyx config <module> --set key=value` on a key that takes either one model-spec or a per-round list writes the shape the value spells: a YAML flow list becomes a real list, any other value a string.
A list value `--set` cannot write as a list is refused, never written as a string.

## Scope

- `internal/yamlengine`: `SetValues` and `SetSparse`, the two write paths behind `--set` (full base layer and sparse worktree/local layers).
- The keys affected are those a module declares as open-map paths whose value is not a mapping.
  In loom these are `review`, `fix`, `fan_review`, `discussion_advisors` and each segment's `*_review` and `*_fix`; loom already declares them through `loomengine.ConfigOpenMaps`.
- `lyx config --help` text in `internal/configcli` and the open-map section of `internal/yamlengine/doc.go`.
- Out of scope: model-spec grammar checks on write (loom's load-time validation and the local layer's `CheckValues` stay the checks), removing an entry, and any change to mapping-valued open maps such as board's `labels`.

## Decisions

### A non-mapping open-map path takes a scalar or a flow list at the path itself

Decision: when a `--set` key is exactly a declared open-map path P and P's current value (the carried file value, else the template's) is a scalar or a list, the value replaces P whole.
A value whose first non-blank character is `[` is parsed as a YAML flow list and written as a list; any other value is written as a string scalar.
The new node keeps the replaced node's comments.

Rationale: open-map paths are the keys whose shape the repository owns, and loom already declares its scalar-or-list model keys that way, so the rule needs no new registry field.
Deciding by the value's spelling, not the current node's shape, lets `--set` move a key from a scalar to a list and back, which today fails in one direction and writes a string in the other.
No model-spec starts with `[`, so a leading `[` never misreads a scalar.

### A list value that is not a flow list of scalars is refused

Decision: a value starting with `[` that fails to parse as YAML, or parses to anything but one list of scalars, is refused with an error naming the key and a quoted-entry example such as `["opus[medium]", "sonnet[high]"]`.
Nothing is written.

Rationale: the board entry asks that a value `--set` cannot write in the key's shape be refused; YAML forbids `[` and `]` inside an unquoted flow-list entry, so a bracketed model-spec entry must be quoted, and the refusal names that way forward.

### Mapping-valued open maps keep their behaviour

Decision: when P's current value is a mapping, the key P itself stays outside the set path as before; `P.<name>` keeps its scalar-entry behaviour.

Rationale: the bug concerns scalar-or-list keys only; a mapping-valued open map has its own `--set` contract.

## Constraints

- `PATTERN-config-layers`: only `configengine` reads and stacks layer files; the change stays in `yamlengine`'s byte-level set operations that `configengine.Set` already calls.
- `PATTERN-comment-line-breaks`: Go comments one sentence per line.
- `PATTERN-test-economy` and `PATTERN-test-speed`: extend the existing `SetValues` and `SetSparse` table tests rather than adding parallel suites.
- Docs in the same commit: `internal/yamlengine/doc.go` open-map section and `lyx config --help`.

## Auto-mode assumptions

- No interview: a darn run holds none, and the board entry fixes the wanted behaviour.
- The unquoted form `[opus[medium], sonnet[high]]` is refused with the quoted form as the way forward rather than parsed by a lyx-specific list grammar.

## Open risks

- A hand-written value starting with `[` meant as a literal string at a non-mapping open-map path would now be parsed as a list; no current key takes such a string.

## Acceptance criteria

- `--set 'plan_review=["opus[medium]","sonnet[high]"]'` writes `plan_review` as a list of those two entries, on the full base layer and on a sparse layer.
- `--set 'plan_review=[opus[medium], sonnet[high]]'` is refused with an error naming `plan_review` and the quoted form, and the file is unchanged.
- `--set plan_review=opus[high]` on a file whose `plan_review` holds a list writes the scalar `opus[high]`.
- `--set review=sonnet[high]` on a scalar key still writes a scalar, and `--set 'review=[]'` writes an empty list.
- A mapping-valued open map's `P.<name>` entry behaviour is unchanged.

## Notes for the plan writer

- `SetValues` mutates the template tree after `carryOpenMaps`; the per-pair loop is where the new branch goes, before the sequence and leaf branches.
- `SetSparse` writes into the existing sparse root through `setEntry`, which already keeps the old node's comments.
- `replaceSequence` holds today's flow-list parse for list keys.
