No findings.

**F1** (sites under `OperatorChangeConstant`): all three `add(..., OperatorChangeConstant, ...)` sites are at `internal/testcover/mutant.go:707`, `:738` and `:745`.
- The package-level variable value site at `:707` calls `markConstantIntegerExpression` through its new `BinaryExpr` case.
- The function-body literal site at `:738` is covered by the call at the top of the body's `BinaryExpr` case.
- The function-body constant-use site at `:745` is covered by the same call, because the `Ident` is marked fixed.
- A constant expression nested inside a larger non-constant expression, such as `x + (1<<63)/2`, is handled when the inner `BinaryExpr` is visited. The parent is always visited before its children, so the marks land before the leaves are reached.

**F2** (tests under `mutantBuildRounds`): the new nested subtest is the only caller-level test of the cap.
- It fails if the cap wiring is reverted: the old failure at the cap gives a non-zero exit code.
- It fails if `wholeFiles` is never passed: round 5 would drop only the line-29 mutants, so `Unbuildable` would be below 21.

**F3** (rows under `lymMutantLine`): the new rows cover both branches the review named.
- A `=` before `: ` is covered by the `a=b: c` row.
- A matched separator followed only by blanks, falling back to the indent, is covered by the `key:  ` row.

**F4** (the cap's doc): two other places state the cap, and both already agree with the reworded constant doc. These are the `buildMutants` doc at `internal/gatecli/redundantmutants.go:391` and `internal/gatecli/doc.go:1190`.

Tool calls made: 1.
- `grep` for `mutantBuildRounds`, `line-only`, `five builds` and `OperatorChangeConstant` in the non-test `.go` and `.md` files, to enumerate every change-constant site and every restatement of the cap. My context held only the files I had read, not a repo-wide search.
