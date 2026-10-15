# Relevant code

- `contracts/stencils/webster/webster-body-implementer.md` — the implementer body whose step 2 gets the deviation rule and whose guard-step paragraph has the dangling clause
- `internal/planglyph/scope.go` `ScopeGuard`, `targetGlyphUnion`, `targetsCover` — the comparison D1 widens to file and package targets
- `internal/planglyph/donecheck.go` `resolveKeyFor` — keys a target in the union, a handle by its glyph, every other ref as-is
- `internal/planparser/targetdirs.go` `RefScope`, `RefCovers` — map a target to its file or package directory and tell member refs apart
- `contracts/stencils/webster/webster-fragment-guard-fix.md` — the scan-error fix, a sanctioned deviation
- `contracts/specs/loom-plan-spec.md` — its deviation union paragraph is stale against the guard
- `contracts/stencils/webster/webster-template-master.md` — reads the digest's guard-filled `deviations`; unchanged
- `contracts/stencils/webster/webster-prefix-fork.md` — composes the body for the fork prompt
- `internal/websterengine/recordbatch.go` `postBatchChecks` — turns guard findings into the digest's deviation list
- `internal/websterengine/doc.go` — the fork-return contract and the deviation list's informational posture
- `internal/websterengine/report.go` `Report` — the fork report's two fields, older `deviations` key ignored
