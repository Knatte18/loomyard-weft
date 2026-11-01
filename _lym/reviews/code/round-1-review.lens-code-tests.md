```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour: pinning test**

1. `NewCleanGate` passes a clean tree and an ignored file, and fails on an untracked or a modified tracked file, naming the path. Pinned by `TestCleanGate` (`internal/loomshed/cleangate_integration_test.go:15`), whose four rows cover it.
2. `UncommittedFindings` text (commit what belongs, delete the rest). Pinned by `TestCleanGate` (finding substring `Commit each one that belongs to the change, delete the rest`). The fixer test asserts `delete the rest` as well.
3. `NewCleanGate` returning an error when `git status` fails. No test pins it.
4. The `"clean"` name resolves in `resolvePlainGateClosure`. Pinned by `TestResolveGateSpec_ValidOneEntryPerVocabularyName` (`entries_gate_test.go:62`).
5. The unknown-name error lists `clean`. Pinned by `TestResolveGateSpec_UnknownNameListsTheLegalValues` (`entries_gate_test.go:339`).
6. `"clean"` refuses a relative `WorktreeRoot` through `requireAbsRoot`. No test pins it.
   - The `"verify"` gate has one: the loop at `entries_gate_test.go:348` covers `VerifyCommand`, `WorktreeRoot` and `VerifyDir`.
   - The `"discussion"` gate has one: the row at `entries_gate_test.go:260`.
   - The `"clean"` gate has none of these.
7. The clean gate is listed first on the Darn-Write and darn Code-Burler-Fix rows, and the sampler Code-Burler-Fix row. Pinned by `TestDarnVerifyBudget` (`internal/loomrecipe/darnrecipe_test.go:288` and `:320`).
   - It compares each row's gates as a map, so it pins presence and attempts, not order.
   - Which recipe the second subtest loads is unconfirmed (see the note under 8).
8. The clean gate on the tapestry Code-Burler-Fix row (`tapestry-recipe.yaml:556`). No test found.
   - `TestDarnVerifyBudget` has no tapestry row.
   - `NameCodeBurlerFix` appears in no tapestry-specific test in the grep output.
   - Not confirmed: whether the second `TestDarnVerifyBudget` subtest is the sampler recipe.
9. Gate ordering, clean before verify. No test pins it, since the map assertion in item 7 ignores order. The behaviour is that a stray file is re-prompted before verify runs.
10. The fixer re-prompts on a dirty tree with a turn left. This charges the turn and carries the findings, and a clean next turn ends fixed. Pinned by the `TestFixerRun_Outcomes` row "a dirty tree with a turn left re-prompts…" (`fixerrun_test.go`).
    - It asserts `wantSpent` equal to `fixerTestCap`, `wantTurns` 2, and the findings via `fakeFixerSeats.findings`.
11. The dirty end when no turn is left, with `rec.Spent >= cap`. Pinned by the row "a dirty tree after the last turn left is the dirty end".
    - It asserts `wantSpent`, the reason containing `(dirty)`, and the `lym loom resume` way forward.
12. `verifyGate.dirtyPaths` returning an error when git status fails, now reached from `judge`. No test found.
    - The grep shows `dirtyErr` only in `battenshed/land_test.go` and `finalize_test.go`, not the fixer test.
    - `clean` still had its own error path before the refactor, so this is unchanged behaviour reached through a new caller.
13. `dirtyPaths` with a nil `dirty` seam returns none. No test found. The seam is set by default in `newVerifyGate`.

**Findings**

- **F-tests-1: MEDIUM, `scope`.** `internal/shedrecipe/entries_gate_test.go` (cf. `:156`, `:260`, `:348`). The `clean` gate's `requireAbsRoot("WorktreeRoot")` refusal has no row, though its sibling gates have one. The `verify` loop at `:348` takes a field list, so adding `clean` as a row there would cover it.
- **F-tests-2: MEDIUM, `scope`.** `contracts/recipes/tapestry-recipe.yaml:556`. The clean gate on the tapestry Code-Burler-Fix row is covered by no recipe test I could find. The darn and sampler rows are covered by `darnrecipe_test.go:288` and `:320`. The tapestry row has no counterpart, so deleting its `clean` entry would break nothing.
- **F-tests-3: LOW, `scope`.** `internal/loomrecipe/darnrecipe_test.go:288`. Gate order, with clean listed first, is not asserted, because the `want` is a map. If order is the contract, as the recipe comments and `loomshed/doc.go` say ("lists it before its verify gate"), a reordering would pass.
- **F-tests-4: LOW, `scope`.** `internal/loomshed/cleangate.go:20` and `internal/landingshed/fixerrun.go:253`. The git-status failure path has no test: `NewCleanGate` returns a wrapped error, and `judge` returns the error from `dirtyPaths` without charging a turn or halting. Both are a trust boundary on git.

No redundant test found. The new `TestCleanGate` has no existing counterpart. The old fixer "dirty" row was replaced, not duplicated.

**Tool calls**
1. A grep for `"clean"`, `NameCodeBurlerFix`, `dirtyErr` and `dirtyAt` in test files. The manifest did not show which tests cover the recipe rows or the dirty-error path.
2. A grep for `WorktreeRoot` in `entries_gate_test.go`, plus a search for tests referencing `tapestry-recipe`. The manifest did not show the relative-root refusal tests or any tapestry recipe test.
