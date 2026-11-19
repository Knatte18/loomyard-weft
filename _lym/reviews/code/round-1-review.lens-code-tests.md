```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, I named the test that pins it or recorded a gap.

**Per behaviour, the test that pins it**

1. `boardengine.ValidateEntrySlug`, the length limit plus the root reservation (`internal/boardengine/task.go:409`). The existing slug-limit and root tests in `task_test.go`, `store_test.go` and `owners_test.go` pin it through `NewTask`. `TestRaddleBoardContext` also pins it through the closure: a 32-character slug is admitted, a 33-character slug is refused, and `root` is refused. The mutation report has every mutant killed. Pinned.
2. `NewTask` now delegates to `ValidateEntrySlug` (`task.go:430`). The existing `NewTask` and upsert tests pin it; reach lists 93 tests. Pinned.
3. `raddleBoardContext.ValidateSlug` runs the entry rule and then the pair rule (`internal/loomcli/raddlecontext.go:85-90`). The `TestRaddleBoardContext` rows pin it: `part-a` is admitted, the 32 and 33 boundary is checked, and `root` and `part/a` are refused. The `part/a` row pins that the pair rule is applied. The junction-name argument is not pinned. Finding F1.
4. The `fabricengine.LoadConfig` failure path inside `raddleBoardContext` (`raddlecontext.go:61-64`) has no test. This is most likely the mutation report's single uncovered changed block. Finding F2.
5. `checker.slugs` reports a refused part slug as `slug-invalid` (`internal/raddleparser/check.go:506-510`). Pinned by the `TestCheck/SlugTheBoardRefuses` row.
6. `slug-invalid` on a project slug. Pinned by the `TestCheck/ProjectSlugTheBoardRefuses` row.
7. A nil `ValidateSlug` checks nothing. Pinned by the `TestCheck/NoSlugRuleChecksNothing` row.
8. The `slug-invalid` detail carries the board's reason and the way forward. Pinned by the `TestCheck_DetailNamesTheWayForward/SlugInvalid` row, which asserts `slug "refused-slug" is refused`, `pick a slug the board admits` and `lym loom raddle-propose`.
9. The `depends-note` way-forward text, which the rewrite carried over. Pinned by the `TestCheck_DetailNamesTheWayForward/DependsOnNote` row. The base test's assertions all survive unchanged: exactly one detail and the same three substrings. Nothing was loosened.
10. Spec and `doc.go` prose. Exempt as static prose.

**Findings**

- **F1 — LOW — design — `internal/loomcli/raddlecontext_integration_test.go:112-125`**
  - **Summary:** No row checks that the pair rule receives the hub's junction names (`fabricConfig.Dirs()`). The closure could pass `nil` and every row would still pass, because `part/a` is refused by the separator rule whatever the junction list holds.
  - **Fix:** Add a row for a slug equal to one of the forged hub's configured pathspec junction names, wanting it refused.
  - **Evidence:** `raddlecontext.go:89`; the separator check in `fabricengine/slug.go` comes before `IsReservedHubName`.
- **F2 — NIT — design — `internal/loomcli/raddlecontext.go:61-64`**
  - **Summary:** No test reaches the new `fabricengine.LoadConfig` error return. The line only passes the error up unchanged, so the gap is of little consequence. It is the mutation report's one uncovered changed block.
  - **Evidence:** the mutation report reads "1 uncovered changed blocks"; `raddlecontext.go:61-64`.
- **F3 — LOW — consistency — `internal/raddleparser/check_test.go:186`**
  - **Summary:** The redundancy report lists `TestCheck_DetailNamesTheWayForward` as contained by `TestCheck`. `TestCheck` compares only finding names (`check_test.go:173-180`) and never the `Detail` text, so this test pins something coverage cannot see.
  - **Fix:** Keep the test and mark it `//testtiming:keep pins each refusal's way-forward detail text` directly above its `func` line. Neither HEAD nor the base carries that marker.
  - **Evidence:** the TestCheck loop at `check_test.go:173-180`; grep found no `testtiming:keep` in HEAD or at `13cebe3d`.

**Tool calls**

1. One grep:
   - looked for `testtiming:keep` in `check_test.go` at HEAD and at the base, to settle F3;
   - listed the boardengine test files that assert the slug-length and root refusals, to confirm behaviours 1 and 2 are pinned;
   - checked whether the context test forges any pathspec junction name, to confirm F1.

   None of this was in my context.
