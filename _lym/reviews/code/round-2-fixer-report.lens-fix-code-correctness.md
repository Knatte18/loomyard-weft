```yaml
lens: fix-code-correctness
outcome: reported
```

Checking the F12 change set for code correctness: no findings.

I traced the one code change, the `//lym:only upkeep` line and reworded doc comment on `TestCSharpFixtures_CompileWithDotnet`:

- **`--gate webster` or another non-upkeep key:** `withOnlyTests` puts the name in `held`. The listed tier runs `-skip` it, and `withoutHeld` removes it from the guard step.
- **`--gate upkeep` with tier 1 listed:** the name goes into `marked[1]`, and the guard step runs it. The `*.cs` watch marker cannot drop it, because a marked scope has non-nil guards and never calls `scanGuards`.
- **`--gate upkeep` without tier 1:** `markedScope` refuses and names the test. The hub already needs tier 1 listed for the other upkeep tests.
- **No `--gate`:** nothing is held, so the watch and guard behaviour is unchanged.

The fix changed no signature and added no test. The new PATTERN sentence about whole gates matches this behaviour.

Tool calls: none.
