# Select an importing package's tests from its coverage map, or name the miss that runs it whole

`coverage-test-select` · Task · feature, gate, testkit, darn

Decision record: [Sketch](decision-coverage-test-select.md)

Depends on: [`testmap-cross-package`](design-testmap-cross-package.md)

Filed from the Sketch `gate-scope-by-coverage-map`.

Takes: testmap-cross-package's map shape, the working tree and fabricengine.Fabric.CodeFileAt for file content at the map's Commit
Delivers: for one importing package's map: the tests selected by changed functions of linked packages per D4, or a D5 miss with its reason (map unreadable or unmarked, Commit unreadable, own files changed, non-body change, //go: line, method set, added init, predeclared name, parse failure, non-Go file change with the go:embed carve-out from inert_paths)
Exports: a selector in internal/testcover returning the selected test names or a miss reason, which coverage-gate-scope calls per importing package

The part changes these packages:

- internal/testcover (about 900 changed lines)
