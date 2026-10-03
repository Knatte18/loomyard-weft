# Card 2 — testkit-scan-on-scankit

**Edit:**
- `internal/testkit/enforcement_test.go`
- `internal/testkit/doc.go`
- `CONSTRAINTS.md`
- `docs/overview.md`

**Uses:**
- `internal/testkit/scankit/scankit.go`

**Intent:** Rebuild the Testkit scan on `scankit` and restate the Testkit Invariant per D19, except its two `tmuxkit` clauses, which land with `tmuxkit` in card 30.
`TestEnforcement_TestkitInvariant` takes its root, walk and floor from `scankit` and keeps its three rules and denied-import set verbatim;
its `kitDeniedImports` map becomes a one-entry-per-line list (D8).
It gains one rule: `scankit` imports the standard library only, asserted with `scankit`'s import-allowlist assertion over `internal/testkit/scankit`.
Prove the rebuilt scan by planting one violation per rule (a non-test file outside the kits importing a kit, and a stdlib-external import in `scankit`), watching each fail and removing the plant.

`CONSTRAINTS.md`'s Testkit Invariant changes its scope from "a seam faked in two or more packages" to shared test support (fakes, builders, fixtures and the scan harness) used by two or more packages, gains the clause that `scankit` imports the standard library only so every package's tests can import it, and widens the review-discipline sentence: `scankit` asserts scan results, while `plankit` and `stencilkit` assert nothing beyond `t.Fatalf` on their own setup.
`internal/testkit/doc.go` states the same scope.
`docs/overview.md`'s Tests passage names `scankit` as the harness every invariant scan runs on, and its sentence placing `internal/testkit`'s kits "for seams faked in more than one package" is restated to the same new scope.

**ImpactSummary:** One scan file rebuilt and one invariant restated; no production code and no other test changes.
