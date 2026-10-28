```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, I looked for what calls, reads or asserts it outside the touched lines.

**Per change, the set found**

1. `baseLineEndPairs` is replaced by `newBaseBreaks` and the `baseBreaks` type.
   - The set is `findNewWrappedBreaks` at `lint.go:286`, the one caller, which the diff updates.
   - A repo-wide grep found no other reference to `baseLineEndPairs`, and `lint_test.go` reaches the lint only through `findNewWrappedBreaks`. Covered.
2. `findNewWrappedBreaks` now calls `endsSemanticBreak` first and `carriesOver` second.
   - The set is `Lint` (`lint.go:57`), the `lint-comments` verb (`internal/loomcli/lintcomments.go`), the card-gate callers (`internal/websterengine/cardgate.go`, `internal/loomshed/verifygate.go`) and the `commentlint` tests.
   - Their signatures and return shape are unchanged. The swapped order of the two checks changes no result, because both are pure and both must fail for a break to be flagged. Covered.
3. The behaviour change is that a rename-only break is no longer flagged. The docs that describe it are:
   - `internal/commentlint/doc.go:10-15`: updated in the diff.
   - `docs/overview.md:320`, which describes the module as "fixed-column wraps in the `//` comment blocks a diff creates".
   - `docs/overview.md:549`, which describes `lint-comments` as the fixed-column-wrapped breaks a diff creates.
   - The `lint-comments` cobra `Long` help (`internal/loomcli/lintcomments.go:33-40`).
   - All three still read correctly, because none of them states the base word-pair rule. Covered.
4. The stencils are `contracts/stencils/loom/loom-rubric-code-review.md:70` and `contracts/stencils/webster/webster-body-implementer.md:87`.
   - They only name the verb, and the diff changes no stencil. Nothing is affected.
5. `PATTERN.md` and `pattern/`:
   - The diff adds no cross-cutting invariant and declares no PATTERN change, and `pattern/PATTERN-gate-slot.md` only mentions the lint as a gate step.
   - The commit message carries no `Pattern-Departure:` trailer, and none is needed because no PATTERN entry forbids the change.
   - The tier-0 rows go into the existing table, as `PATTERN-test-speed` asks. Covered.
6. The docs-land-in-the-same-commit rule is met: `doc.go` changed in commit `28b90bf4`. `docs/overview.md` needs no edit because the module table and execution stack are unchanged.
7. Commit messages: both end with the attribution line, and the subjects match the diff. The second commit, `a92cc93b`, only re-breaks the new comments.

**Findings:** none.

**Tool calls**
1. Grepped the repo for `lint-comments`, `commentlint` and `word pair` to find callers and docs, because the manifest did not list them.
2. Read the relevant lines of `docs/overview.md`, `lintcomments.go`, the stencils and `doc.go`, to check whether any of them states the base word-pair rule.
