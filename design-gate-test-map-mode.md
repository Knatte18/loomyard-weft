# Build the coverage maps with lym gate test --map, one process per test in parallel, and retire lym gate testmap

`gate-test-map-mode` · Task · enhancement, gate, testkit, dobby, docs, pattern · running · Drift-Verify

Decision record: [Sketch](decision-gate-test-map-mode.md)

Filed from the Sketch `gate-builds-testmap`.

Takes: the per-package measure loop of internal/gatecli/testmappackage.go and the testcover map reader, writer and profile reduction as they stand
Delivers: lym gate test --map over tiers 0 and 1 writing each package's map whole, with the verb lym gate testmap, its planning, audit, build lock, wait, --max, run-record ordering and lym build removed, the testcover code only that planning reached deleted, and every live doc, background and lym gate redundant refusal naming the flag
Exports: the --map flag of lym gate test and its detached-call key word; no new Go API outside internal/gatecli

The part changes these packages:

- internal/gatecli (about 3700 changed lines)
- internal/testcover (about 2050 changed lines)
- internal/gateslot (about 20 changed lines)
- internal/dobbyengine (about 5 changed lines)
- cmd/lym (about 2 changed lines)
- docs (about 20 changed lines)
- pattern (about 60 changed lines)
- README.md (about 20 changed lines)
