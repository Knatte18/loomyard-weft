```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it.

**Per behaviour: the pinning test**

The diff adds no test function and changes no assertion. The behaviours it changes are fixture start states, and each one is pinned by the tests that consume that fixture:

- `anchorTemplate` and `newAnchorHub`: the prime warp tracks `sub/` and `other/` in unpushed commits. The tests in `TestAnchorChange` that call `newAnchorHub` or `newIntegrateHub` pin it (`anchorchange_integration_test.go:215`, `:496`, `:632`, `:713`, `:789`). One example is "A failure before an integration moves the branch leaves the pair wired where it was" at `:712`. Breaking the start state would fail the wiring and HEAD assertions there.
- `originTemplate`, `originHubOf` and `newOriginHub`: the records branch is published with its upstream, and the branch names are read from the copy. The `ls-remote` and `fetch` call-count assertions in `TestReconcileOrigin_Judge` pin it (`originreconcile_integration_test.go:222`, `:298`). So do the fast-forward verdict rows in `TestReconcileOrigin_Moves` (`originreconcilemoves_integration_test.go:154`).
- `originTaskTemplate`, `buildOriginTask` and `newOriginTaskHub`: the pair "task" exists with its upstream set.
  - "An origin that moved nothing is read by two ls-remote calls and no fetch" (`originreconcile_integration_test.go:274`) expects every branch level, which requires the upstream.
  - The diverged, rewritten and gone rows (`:374`, `:460`, `:476`, `:502`) and `newRewoundBranch` (`:804`) pin the pair's worktree and branch.
- `movesTemplate` and `newMovesPair`: the "task" run record names Land, and the pair's records branch is published. The ~20 `newMovesPair` sites pin it, among them the `TestReconcileOrigin_Moves` rows at `originreconcilemoves_integration_test.go:209`, `:243` and `:293`, and the `droplocal` and `worktreeseam` tests.
- `endedTemplate`, `arrangeEnded` and `endedHub`, with `originBoard` now run on the copy:
  - The ended-set rows pin the start state. They are "An entry done on origin with a local pair is handed to the teardown seam once…" (`originreconcileended_integration_test.go:126`), the Finalize and Raddle rows (`:186`–`:229`), and the Busy table at `:303`.
  - The two-pair teardown-failure test pins `arrangeEnded` with several slugs (`:323`).
  - The `originBoard`-on-copy behaviour is pinned by the same rows, because they need the origin's "done" board to reach `ended`.
  - The Moves row at `originreconcilemoves_integration_test.go:643` also covers `endedHub`.
- The `testing.TB` signature changes (`originPair`, `writeStatus`, `publishBoard`, `publishBranch`, `boardFileOf`): these have no behaviour of their own and are exercised through the callers above.
- Template build contract (the `TB` refusals and the board at upstream tip): this belongs to the `hubforge` registration's tests, outside this diff. The new builds call none of the refused TB methods, and every board commit in a build is pushed.
- Seam ordering (each copy taken before the process-global hook is installed): checked by reading rather than by a test, because it is review discipline. Every `SetOriginCallHookForTest`, `SetOriginVerdictHookForTest` and `SetAnchorFailHookForTest` site I read comes after its copy.
  - The anchor sites are `anchorchange_integration_test.go:496`, `:529`, `:632`, `:657`, `:713` and `:745`.
  - The origin sites are `originreconcile_integration_test.go:234`, `:278`, `:306`, `:560` and `:597`.
  - The moves and ended sites are `originreconcilemoves_integration_test.go:169` and `:462`, and `originreconcileended_integration_test.go:166`.
- `InstallReplay`: `boardkit.InstallReplay` still sits at the top of the Judge, Moves and Ended tests, before any copy. In `boltreplayers_integration_test.go:103`, `newMovesPair` runs before `WithoutBoltReplayersForTest`, so the build never runs under an emptied registry.

**Findings**

None. Each changed behaviour has a covering test, and no new test duplicates an existing one or adds an uncovered behaviour. The `Add` calls that moved into templates are fixture steps, not behaviour under test. The per-row and one-off slugs (`first`, `second`, `stopped`, `active`, `unrecorded`, `mine`, `plain`, `ghost`) still call `originPair` in the test.

**Tool calls**

I made no tool calls; the inherited context held everything the lens needed.
