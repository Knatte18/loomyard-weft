# testmap records each test's coverage across every in-module package its binary links

`testmap-cross-package` · Task · feature, gate, testkit, darn · running · Drift-Await

Decision record: [Sketch](decision-testmap-cross-package.md)

Filed from the Sketch `gate-scope-by-coverage-map`.

Takes: testmap and the redundancy report as they stand once testmap-upkeep's design has landed
Delivers: maps whose blocks span every linked in-module package, with per-linked-package file hashes (go:embed targets in subdirectories included), each test's spawn-scan verdict and a format marker holding Commit and BuiltAt; dirty checks over every linked package's files, read fresh before the build and after the last test; the redundancy report and mapDrift unchanged in verdict
Exports: testcover.Map's linked-package field, spawn verdict per test and format marker; BlockKey carrying an import path for another package's block

The part changes these packages:

- internal/testcover (about 300 changed lines)
- internal/gatecli (about 500 changed lines)
