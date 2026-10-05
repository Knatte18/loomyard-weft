# Card 1 — yamlengine carries declared open maps whole

**Edit:**
- `internal/yamlengine#Reconcile`
- `internal/yamlengine#MissingKeys`
- `internal/yamlengine#FillMissing`
- `internal/yamlengine#fillMapping`
- `internal/yamlengine#SetValues`
- `internal/yamlengine/reconcile_test.go`
- `internal/yamlengine/fill_test.go`
- `internal/yamlengine/set_test.go`

**Create:**
- `internal/yamlengine/doc.go`

**Intent:**
Add open maps (D3) to the four template-shaped operations.
Each of `Reconcile`, `MissingKeys`, `FillMissing` and `SetValues` gains a trailing variadic `openMaps ...string` parameter: dotted key paths whose value is an open map.
Variadic keeps every existing caller compiling unchanged, and an absent argument keeps today's exact-key behaviour on every mapping.

For a declared path P:

- `Reconcile`: when existing holds key P, the template's value node at P is replaced by existing's value node whole, whatever its kind (a legacy list, D2, is carried as a list).
  No path equal to P or under it (`P.x`, `P[0]`) is reported in added or removed.
  When existing lacks P, the template's value stays and added reports P itself, never its leaves.
- `FillMissing`/`fillMapping`: a key at P that the file holds is skipped before any shape check, so a sequence at P against a mapping template is no shape mismatch; a missing P is still appended whole from the template.
- `MissingKeys`: a template leaf at or under P is satisfied by the presence of key P in existing, whatever its value.
- `SetValues`: a key `P.<name>` is known for any `<name>`.
  When the working tree's value at P is a mapping, the pair sets `<name>` to the scalar value, appending the key when absent and giving the mapping block style so an empty `{}` template value is written as a block.
  When the value at P is a sequence, `SetValues` returns an error naming P and saying to rewrite P as a map of name to description; it never converts the list.
  `Known` lists each open map as `P.<name>`.

Bound: only declared paths change behaviour; an undeclared mapping still reports added and removed keys and still refuses an unknown `--set` key.
What it lets through: `MissingKeys` no longer inspects the value at P, and a mistyped `<name>` under P creates a new key instead of refusing; the shape check on P is the declaring module's decode.

Tests, untagged: for a declared open map, reconcile keeps every existing key under P and adds none from the template, and reports nothing added or removed; a missing P is filled whole and reported as P; a list at P is carried whole; `MissingKeys` reports nothing under a present P; `SetValues` adds `P.x` to a map, refuses `P.x` on a list, and still refuses an undeclared nonexistent key; an undeclared mapping reports added and removed exactly as today.

`doc.go` gains the package doc for `yamlengine`: what reconcile, fill, missing-key detection and set do, and the open-map rule above.

**ImpactSummary:** yamlengine's four entry points gain an optional parameter; existing callers compile and behave unchanged.

**Commit:** 1: yamlengine carries declared open maps whole
