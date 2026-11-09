```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fix-code-correctness lens over the F1 test `TestKeyListWayForward` and the F2 comment reword in `internal/configreg/configreg_test.go`.

Trace of `keyListWayForward(list, got, want)` against each added row, with `list = "wantList"`:

- **AddOnly**, got `[a b]`, want `[a]`:
  - `keysOutside(want, got)` returns `[b]`, so the step is `add ["b"] to wantList`.
  - `keysOutside(got, want)` returns nothing.
  - The result is `if the classification is meant, add ["b"] to wantList in this test`, which matches the row.
- **DropOnly**, got `[a]`, want `[a b]`:
  - The add direction finds nothing.
  - The drop direction returns `[b]`, so the step is `drop ["b"] from wantList`.
  - The result matches the row.
- **AddAndDrop**, got `[a c]`, want `[a b]`:
  - The add step is `add ["c"] to wantList` and the drop step is `drop ["b"] from wantList`.
  - They join with " and ", which matches the row.
- **OrderOnly**, got `[b a]`, want `[a b]`:
  - Both directions are empty, so `steps` is empty.
  - The result is `sort wantList in this test`, which matches the row.

`fmt.Sprintf("%q", []string{"b"})` prints `["b"]`, so the expected strings are right. The gate passed (tier0, 177 targets, 0 failed) with the new test included.

Each row would fail with a defect in place:
- Swapping the arguments of `keysOutside` swaps add and drop, which fails AddOnly, DropOnly and AddAndDrop.
- Dropping the `len(steps) == 0` branch fails OrderOnly.
- A wrong list name in the format string fails every row, because the rows compare exact strings.

Callers: the signature of `keyListWayForward` is unchanged. The three call sites in `TestModules_EveryTemplateKeyClassified` still hold.

The F2 comment reword changes only comment text.

The new test sits inside the same file and package as the helper, and calls `t.Parallel()` after `tiergate.Require`, as the file's other tests do.

no findings

Tool calls made: none. My context already held the changed file, the review, and the rubric.
