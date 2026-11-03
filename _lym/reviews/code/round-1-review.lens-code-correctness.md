```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running a concrete input through each branch the diff adds or changes.

**Per-branch list**

| Branch | Case run | Result |
|---|---|---|
| `SetValues` whole-map branch (`set.go:221`) | `capture={command: x, timeout_min: 30}` onto template `capture: {}` | Correct. `openMapEntryNode` gives a mapping with `Style = 0`. `setEntry` replaces the node and keeps the old comments. |
| same | `{}` over an existing populated map | Correct. Writes an empty mapping, which marshals as `{}`. |
| same | `""`, `"x"`, `"[a]"` | Correct. Each parses to a non-mapping node, so the refusal names the key and `P.<key>`. |
| same | malformed `{a: ` or text after `}` | Correct. `openMapEntryNode` refuses, naming the key and the quoting rule. |
| same | dotted P (`buttons.task`) with a mapping template | Correct. `mappingAt` walks or creates the parent. |
| same | error on the second of two pairs | Correct. The tree is mutated in memory only and nothing is written. |
| `SetSparse` whole-map branch (`layers.go:258`) | `capture={...}` onto an empty or comment-only existing file | Correct. `existingRoot` is created first, `parent` is computed, then the branch replaces the entry. |
| same | P whose template value is a scalar or list | Correct. `scalarOrListPaths` at `layers.go:250` runs first, so this branch is not reached. |
| `setWholeOpenMap` | shape given, key the shape lacks | Correct. The error names `capture.timeoutt` and lists the keys held. Nothing is set. |
| `setWholeOpenMap` | shape nil | Correct. Any key is accepted, and a nested mapping or list is written as spelled. |
| `mappingShapeError` | `{t0: {nope: 1}}` over `tiers` (`Entries` of `Fields`) | Correct. Recursion through `Entries`, then `Fields`, names `tiers.t0.nope`. |
| same | `Text` or single-value shape, or a non-mapping node | Returns nil. This is the F1 finding below. |
| `knownKeys` | P already in `templateLeaves` (empty `{}` leaf) or `templateSequences` | Correct. `slices.Compact` after the sort removes the duplicate. |
| `unknownKeys` | key equal to P | Correct. The key is now known. `P.x` still goes through `isOpenEntry`, so keys that worked before behave as before. |
| ordering vs `shapedOpenMapKey` | key equal to P | Correct. `splitOpenMapKey` needs the `P.` prefix, so a bare P never takes the shaped path. |

**Findings**

- **F1, LOW, design, `internal/yamlengine/set.go:401` (`mappingShapeError`):** the whole-map shape check tests key reach only, not the kind of the value at a key.
  - `capture={command: {a: b}}` or `{command: [x]}` (a `Text` field) passes with nil, and a mapping or list is written into a string field.
  - The per-key form `capture.command=...` goes through `setShapedValue`'s `shape.Text` branch and always writes a string.
  - So the whole-map form is weaker than the per-key form it replaces. The bad file is caught only at the next `LoadConfig`, or by the hub-wide validate-before-commit check.
  - Evidence: `set.go:401` returns nil for `Fields == nil && Entries == nil`; `set.go:470` is the `Text` case.
  - Suggested fix: in `mappingShapeError`, when `shape.Text` is set and the node is not a scalar, return an error naming the key.

- **F2, NIT, consistency, `internal/yamlengine/set.go:221` vs `layers.go:202,258`:** the whole-open-map check runs before the shaped-key check in `SetValues` and after it in `SetSparse`.
  - Today no module declares overlapping open maps such as `a` and `a.b`, so no input reaches the difference.
  - Evidence: `set.go:221-227` against `layers.go:202-207` and `258-263`.

I found no wrong result and no swallowed error in the other branches. In `SetSparse` the whole-map branch repeats the `mappingAt` call already made at `layers.go:246`. That is harmless because `mappingAt` is idempotent.

**Tool calls:** none. The manifest already held `set.go`, `layers.go`, `config.go` and the diff.
