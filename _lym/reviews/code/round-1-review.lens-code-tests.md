```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test row that pins it, or none.

**Per behaviour**

1. `judge` stores the dirty paths on `outcome.dirty` at the dirty end (`fixerrun.go:261`).
   - Pinned by the `TestFixerRun_Outcomes` row "a dirty tree after the last turn left is the dirty end" (`fixerrun_test.go:285-294`).
   - Without the assignment, `listShown(nil)` is empty and the reason reads "after its last turn: ;". The `"after its last turn: a.go;"` substring would then fail.

2. `haltReason` gives `fixerEndDirty` its own cause and way forward (`fixerrun.go:333-335`).
   - Pinned by the same row's `wantReason`: `LandingStopPrefix`, `fakeFixerRunDir`, "commit or remove them on the task branch" and "`lym loom resume`".
   - `avoidReason: "without a usable turn"` pins that the old generic wording is gone.
   - The shared checks after the row also apply: the unjudged and judged reasons are equal, and the reason has the prefix and a "way forward: " part.

3. `fixerEndTimeout` and `fixerEndNoReport` keep the old wording (`fixerrun.go:330-332`).
   - Rows at lines 225, 236, 247 and 265 assert the prefix, `fakeFixerRunDir` and the resume command.
   - No row asserts "ended without a usable turn" or the `(timeout)` / `(no-report)` token.
   - The behaviour is unchanged, so this is not a gap the diff opens.

4. The `reflect.DeepEqual` replacement for the `fixerOutcome{}` comparison is pinned by the subtest "a failed start leaves the record as it was..." (`fixerrun_test.go:410`).
   - It is the only `fixerOutcome` struct comparison. My grep found no other `!=`/`==` use in `internal/landingshed`; the other `fixerOutcome` types are separate types in `gatefixer` and `gatecli`.

5. The `listShown` cap applied to the dirty paths has no row with more than `dirtyPathsShown` paths in the dirty end.
   - The cap is `listShown`'s own behaviour, and the other dirty-tree reasons in `verifygate.go` already use it.
   - A regression to a plain join at this call site would pass. I do not raise it: it would be a per-helper test the existing coverage already implies.

6. The `doc.go` sentence is static prose and exempt.

**Findings:** none.

Tool calls: none; the manifest already held the diff, `fixerrun.go`, `fixerrun_test.go` and the grep of `fixerOutcome` uses.
