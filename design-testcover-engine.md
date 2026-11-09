# internal/testcover: the coverage runner, content block keys and the map record, with testtiming's package-scoped verdict

`testcover-engine` · Task · feature, testkit, gate · running · Preflight

Decision record: [Sketch](decision-testcover-engine.md)

Filed from the Sketch `redundant-test-finder`.

Takes: cmd/testtiming's runner, profile parsing, containment and greedy pass, spawn scan and keep scan
Delivers: internal/testcover holding the moved runner, eligibility rules, spawn scan and keep scan, content block keys (D3), the per-package map record written and read through internal/state at a path internal/lymdirs names (D4); cmd/testtiming judging by the package view with a module-wide column, its flags and -pkg splice unchanged; cmd/lym guard tests re-pointed at the moved files
Exports: the engine's build, run-alone, profile parse, block key, verdict and greedy functions, the spawn and keep scans, and the map record type with its read and write

The part changes these packages:

- internal/testcover (about 1300 changed lines)
- cmd/testtiming (about 300 changed lines)
- cmd/lym (about 10 changed lines)
- internal/lymdirs (about 10 changed lines)
