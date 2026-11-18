```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the searches for F2, F5, F6, F7 and F8. Two sites were left stale: one for F7, one for F2.

- **LOW (F7, missed member of the set):** `contracts/stencils/stencils.go:54`, the doc comment of `DarnFragmentMakeChangeGated`.
  - It still says "The writer and its subagents run no gate in it."
  - It embeds the very fragment whose header was changed, and its next line describes the parent-asked run, so it should name the writer's single measuring run of one test class.
  - The review named the fragment header, `darnengine/doc.go:79` and the `GateFixed` and `DarnTable` comments in `spec.go`. It did not name this fourth site, and a search for "run no gate" finds it.

- **LOW (F2, stale sibling):** `internal/gateslot/doc.go:202-203` and `internal/gateslot/tiers.go:359-360`.
  - They say the gate calls of a tierless hub "render and run nothing" and "run their gates with nothing to run".
  - After the F2 fix, such a call runs the key's lints and can exit 1.
  - The `doc.go` comment on the key's lints is the one a reader of `gateslot` would use.

Rechecked and covered:
- **F5, F6, F8:** every scope-all description I found agrees with the fix. These are `cli.go:73`, `template.yaml:21` and `gatecli/doc.go:254`.
- **`scope.go:104`:** it says "A key of scope all is the --all form". This is still accurate, since the empty scope is applied in `allScope`.

Tool calls (2 of 6 used, plus 2 small reads):
1. Grep for the no-gate phrasings across contracts, internal, docs and pattern. My context lacked the full set of sites that state the writer runs no gate.
2. Grep for the scope-all and whole-tree phrasings. My context lacked every sibling description of scope all.
3. Read `stencils.go` lines 50-58 and grepped for "declares no tier". I needed the exact text of the stencil's doc comment and the tierless wording in `gateslot`.
