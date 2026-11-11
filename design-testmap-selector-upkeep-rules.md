# The coverage selector takes its target, compares own files by function, and misses on a changed environment stamp

`testmap-selector-upkeep-rules` · Task · feature, testkit, gate, dobby

Decision record: [Sketch](decision-testmap-selector-upkeep-rules.md)

Depends on: [`gate-scope-coverage`](design-gate-scope-coverage.md)

Filed from the Sketch `testmap-upkeep-design`.

Takes: coverage-test-select's Selector in internal/testcover as it lands, which reads the working tree only and misses on any own-file change; the map format testmap-cross-package landed (FuncKey function sets, OtherFiles, MapFormat with an EnvironmentStamp); internal/gatecli as gate-scope-coverage leaves it
Delivers: the selector with its target as an argument (working tree or a commit through fabricengine.Fabric.CodeFileAt); own non-test Go files compared by function; own non-Go files, testdata and its Go files included, a miss unless inert_paths exempts them and no go:embed pattern names them; test files compared with every top-level Test function's text removed; outcome and spawn entries read as no recorded test; a differing environment stamp a miss; the stamp holding the toolchain, GOOS/GOARCH, go.mod and go.sum hashes, the mapped tiers' command and environment digest before {parallel}, and the env.yaml variables digest; D3's map fields checked against the landed format and completed where they fall short
Exports: the selector's target argument and its verdict (affected tests or a miss with its reason), which testmap-upkeep-plan's plan and self-check call; the environment stamp builder; the MapTest outcome and spawn fields the selector reads

The part changes these packages:

- internal/testcover (about 650 changed lines)
- internal/gatecli (about 120 changed lines)
