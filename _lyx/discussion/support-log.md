## Interview

Autonomous mode: no operator answered.
Each turn below is a question the board entry and the code left open, with the answer picked on judgment.

1. Read the board entry `plan-gate-checks` (issues #461, #451, #460, note `caller-scan-import-path`) and the related entries `plan-glyph-precision` (done; built `caller-uncovered`, `redundant-file-target`) and `glyph-practice-audit` (note; audits glyph use after landing).
2. Read `internal/planglyph/callercoverage.go`: the scan tokenizes every Go file and matches a method by `.` plus name anywhere, and a package-level member by package clause, so it cannot see aliases or dot imports and cannot tell receivers apart.
   Quarry v0.2.1 exposes no reference or caller API.
3. Q: how to resolve receiver types? Picked go/types through `golang.org/x/tools/go/packages` over the candidate directories (D1).
4. Q: what when the load fails? Picked a per-file, per-identifier fallback to an import-path and name scan, never an infrastructure error (D2, D3).
5. Q: severity of a type-resolved method finding? Picked blocking; name-only method matches stay informational and collapse per subject (D4).
6. Read `internal/websterengine/cardgate.go`: the Go-derived card gate runs `go test -tags integration ./<dir>` from the root for every target directory, which is the unsatisfiable gate of issue #451 when the directory sits in a nested module.
   Picked `go -C <module>` per nested module (D7).
7. Q: should author-written verify commands be checked too? Picked yes, a new blocking planparser check `verify-nested-module` (D8), since the plan-level `## verify:` and a card's `Verify:` can name a fixture package.
8. Q: where does the nested-module set come from? Picked one planparser function, the union of on-disk and plan-created `go.mod` files through the card (D6).
9. Read `loom-rubric-plan-review.md`: the rubric points at the spec's check list only under "Do not flag", so a proposed fix is unbound by it.
   Picked a rubric rule binding every proposed fix to the spec's validation-checks section (D9) and the matching writer-side line in the plan template (D10).
10. Q: the test-budget symptom of issue #451? Read `cmd/lyx/testdata/test-budget.yaml` and the implementer stencil: an untargeted edit is an informational deviation.
    Picked a `CLAUDE.md` line (D11), since the file is this repo's own guard.

## Rejected alternatives

- Hand-rolled loader (`go list -export -deps -test` plus `go/types` with the gc importer): duplicates what `go/packages` does for test variants and cgo, with more code to get wrong.
- `go/importer`'s source importer: type-checks every dependency from source on every gate run, the standard library included, so it is slow, and it needs no less toolchain.
- An AST-only local type inference for method receivers (receiver, parameter and `var` declarations): misses field chains, return values and embedding, so it would keep a long tail of name-only matches.
- Adding a reference API to quarry: another repository, outside this worktree, and quarry's tree-sitter model has no type information to answer receiver questions.
- Failing the gate with `ErrQuarryUnavailable` when the load fails: blocks plans over a toolchain hiccup on a check that before this task was informational for methods.
- Keeping type-resolved method findings informational: leaves a certain uncovered caller to webster's build failure.
- A rate limit (first N matches) on name-only findings: drops matches silently; collapsing keeps them all in one finding.
- Skipping nested-module directories in the caller scan: drops callers of a root subject in a module that imports the root, and every caller of a nested subject, with no other gate building that module.
- Counting a `Move` of a `go.mod` in D6 as a `Delete` of its source: the destination is in `Intent` prose, so the module would vanish from the set rather than move, which is wrong in the other direction.
- Dropping a finding whose obvious fix a check blocks: leaves a defect the checks miss unreported; binding only the proposed fix keeps it.
- Running a nested module's tests through `cd <module> && …` in the card gate: `go -C` does the same with no shell state.
- Validating a reviewer's proposed fixes mechanically before the fixer sees them: a proposed fix is prose, with no form a validator can run.
- Copying the gate's check list (or `redundant-file-target` alone) into the rubric: violates `PATTERN-producer-pointer-rule` and goes stale.
- Treating `cmd/lyx/testdata/test-budget.yaml` as an implicit target in lyx, or naming it in the plan stencil: hardcodes one repository's guard into a generic tool.
- Dropping the test-budget symptom from scope: it cost friction notes on several cards, and a one-line repo instruction is proportionate.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): receiver resolution mechanism — `golang.org/x/tools/go/packages` with go/types over candidate directories; the precise answer, with the Go team's loader.
- Q2 (resolved, auto-pick): adding `golang.org/x/tools` as a direct dependency — accepted; the module already depends on other `golang.org/x` modules.
- Q3 (resolved, auto-pick): load failure behavior — degrade per file and identifier to the import-path and name scan, no infrastructure error; never worse than the pre-task behavior.
- Q4 (resolved, auto-pick): load bounds — `GOPROXY=off`, `GOFLAGS=-mod=readonly`, a 3-minute timeout, logged spawn; a plan gate must not reach the network or hang.
- Q5 (resolved, auto-pick): severity — type- or import-resolved references blocking for methods too; name-only method matches informational, one collapsed finding per subject.
- Q6 (resolved, auto-pick, revised in review round 1): nested modules in the caller scan — scanned through the D3 fallback, since a nested module can hold a caller of a root subject and a nested subject's callers live there.
- Q7 (resolved, auto-pick): card gate for nested modules — `go -C <module> test -tags integration` with module-relative paths, one command per module.
- Q8 (resolved, auto-pick): plan-time check for author-written verify commands — new blocking planparser check `verify-nested-module`, recognizing `go -C` and a literal preceding `cd`.
- Q9 (resolved, auto-pick): nested-module set — one planparser derivation, on-disk plus `Create`-named `go.mod` through the card, minus `Delete`-named ones, a Rename pair counting as both; a `Move` of a `go.mod` is not counted, since its destination is prose, and is a stated bound (revised in review round 2).
- Q10 (resolved, auto-pick): reviewer and the gate — a rubric rule binding proposed fixes, never findings, to the spec's validation-checks section by pointer; the template states the `go -C` form.
  A finding with no admissible fix is deferred by the fixer with the constraining check as its reason and reaches the operator through the judge's escalation (revised in review round 2).
- Q11 (resolved, auto-pick): test-budget symptom — one `CLAUDE.md` line; no lyx mechanism.
- Q12 (resolved, auto-pick): `verify-nested-module` under `language: none` — runs anyway, since it reads `go` commands, not glyphs.
