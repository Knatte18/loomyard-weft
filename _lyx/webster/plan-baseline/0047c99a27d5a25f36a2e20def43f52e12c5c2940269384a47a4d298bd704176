# Card 22 — friction-leaf-prune

**Edit:**
- `internal/batcher#`
- `internal/buildinfo#`
- `internal/friction#`
- `internal/frictionengine#`
- `internal/modelspec#`
- `internal/pattern#`
- `internal/shell#`
- `internal/tokenvocab#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of the friction packages (`internal/friction`, `internal/frictionengine`), the leaf packages (`internal/buildinfo`, `internal/modelspec`, `internal/pattern`, `internal/tokenvocab`) and the registry and seam packages `internal/batcher` and `internal/shell`.

The import-allowlist tests of the leaf packages and of `internal/friction` (`PATTERN-leaf-packages`) keep their closed sets.
`internal/batcher`'s registry tests (`PATTERN-batcher-registry`) and `internal/shell`'s seam tests (`PATTERN-shell-mechanics-seam`) keep their selection and standard-library-only checks.

**ImpactSummary:** Test files of the friction, leaf, registry and seam packages only; no production change.

**Commit:** 22: Prune friction, leaf and seam package tests
