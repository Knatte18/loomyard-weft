```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside its own lines, per change.

**List (per change, set found, covered or not)**

1. **Folded IDs rewritten to cluster IDs** (44 removed IDs, 43 files outside `pattern/` and `PATTERN.md`).
   - Set: Go comments and error strings in `cmd/lym/*_test.go` and `internal/*`, `contracts/stencils/*_test.go`, `contracts/specs/loom-plan-checks-spec.md`, `docs/overview.md`, and `pattern/` cross-references.
   - Result: a repo-wide grep for every removed ID, excluding `docs/research`, `.lym` and `_lym`, finds nothing, so covered.
   - Hits remain only under `docs/research`, which the answer key excludes.
   - Each rewritten citation points at the cluster that holds the rule. I checked these:
     - `PATTERN-pair-remote` in `fabric-boundary` housekeeping.
     - `PATTERN-fabric-boundary` in `pair-remote`.
     - `PATTERN-config-layers` for sparse-hub-base in `hubreconcile/doc.go`.
     - `PATTERN-cli-cobra` for sole-wiring in the `cliwire` tests.
     - `PATTERN-landing` in `parentlock/doc.go`.
     - `PATTERN-run-records` in `friction/doc.go`.
     - `PATTERN-shed-driver` in `shedengine/doc.go`.
     - `PATTERN-lym-files` in `shedengine/doc.go`.
2. **Strings the new IDs are asserted against.** The `planparser` Detail strings and `loom-plan-checks-spec.md` both now say `PATTERN-fabric-boundary`.
   - No test or fixture asserts the old `PATTERN-fabric-vocabulary` string, because the grep finds none. Covered.
3. **Deleted and renamed background files, and links to them.**
   - No markdown link or path to a removed `pattern/PATTERN-*.md` remains. `docs/overview.md` links are updated to `PATTERN-fabric-boundary.md` and `PATTERN-pair-remote.md`. Covered.
   - Files under `pattern/` are 28, and each is named by exactly one overview entry. `PATTERN-github-auth` and `PATTERN-shell-guard-table` have no background link, as before. Covered.
4. **Prose references without the `PATTERN-` prefix**, such as "the fabric-vocabulary entry".
   - A search of `pattern/`, `PATTERN.md`, `docs`, `contracts`, `plugins`, `internal` and `cmd` finds none left.
   - The `pair-teardown` text was rewritten to "fabric-boundary entry".
   - The remaining hit, `internal/lymcwd/enforcement_test.go:627`, is a descriptive comment, not an entry citation.
   - Covered.
5. **Overview shape and `internal/pattern` checks.**
   - `check.go` and `headroom.go` are untouched, so the cap and `HeadroomBytes` are unchanged.
   - The overview is 7532 bytes against a 16384-byte cap, about 46%.
   - It has 30 entries, and all 8 section headings are unchanged.
   - Entry lines use the `PATTERN-[a-z0-9-]+` name, the `(test)` suffix and the background link as `internal/pattern` expects. Covered.
6. **"Declared PATTERN changes landed".** The directive declares none, so there is nothing to check.
7. **Commit message and `summary.md` claims.** I checked each against the diff:
   - 15541 to 7532 bytes: base is 15541, head is 7532.
   - 64 entries to 30: both counts match.
   - 43 files outside `pattern/`: matches.
   - Ten unchanged entries: matches.
   - Not changed here: `_bolt/_lym/specs/loom/loom-plan-checks-spec.md` (deployed copy) and the board entries still cite folded IDs. The summary says so itself, and neither is in the diff.
   - The commit carries no `Pattern-Departure:` trailer and needs none.
8. **Stencil wording.** The diff edits no stencil text, only `*_test.go` comments, so there is nothing to weigh against the answer key.

**Findings:** none.

**Tool calls:** none. My inherited context already held the diff, the old-ID grep, `PATTERN.md`, `internal/pattern`'s `doc.go` and `check.go` hits, and the summary.
