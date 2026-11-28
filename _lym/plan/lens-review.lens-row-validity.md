```yaml
lens: row-validity
outcome: reported
```

Review of the plan's planned test rows through the row-validity lens: whether each row reaches its branch, fails when the code is wrong, and is needed.

**Findings**

1. **BLOCKING — Card 4, D10/D8 create: the create path's pre-check refuses every create.**
   - Card 4's `applyWrite` first reads the entry from `Board.View()`, and "a missing entry or a revision unequal to the request's answers the stale refusal from that view without a write".
   - A create names a slug that does not exist yet, so every create hits the "missing entry" branch and is refused as stale.
   - The `TestServer` ops-table row "create … answers the fresh entry" would then fail.
   - The pre-check also takes the create-of-an-existing-slug case before card 2's in-lock check can see it.
   - Fix: exempt `create` from the view pre-check. Refuse a create only when the slug exists, with that entry's projection, as `Expecting(slug, "")` does under the lock.

2. **MEDIUM — Card 6, `TestRunOpen` / D12: the run-agent row cannot be reached.**
   - The row says "a run agent's strand name is refused with nothing spawned".
   - Card 6 puts `refuseRunAgent` in each verb's RunE before `runOpen`, and step 1 says "`refuseRunAgent` is already passed". So `runOpen` holds no run-agent branch for this row to reach.
   - The behavior is already covered by `TestServeOpenRefusals`.
   - Fix: drop the row from `TestRunOpen`, or move the check into `runOpen`.

3. **MEDIUM — Card 6, `openDeps`: the zero value of the opener-cap seam has no stated meaning.**
   - `runOpen` must choose between "capped at `deps.OpenerCap`" (strand name set) and "waits until the opener exits" (none). The card lists neither `openDeps`' fields nor what a zero `OpenerCap` or an empty strand means.
   - The rows "an orch strand name proceeds" and "success at the cap" depend on that unstated rule. As written, the orch row exercises nothing in `runOpen` that differs from the no-strand row.
   - Fix: list `openDeps`' fields, and state that a zero `OpenerCap` means no cap (wait for exit), which the verb sets only when the strand name is empty.

4. **MEDIUM — Card 6, `TestBoardServeScenario` and `TestServeOpenRefusals`: an inherited `LYM_STRAND_NAME` breaks the rows.**
   - `lymbin.RunWithEnv` appends to `os.Environ()` (`internal/testkit/lymbin/lymbin.go:171`).
   - When the gate runs inside a run agent's session, the variable carries that agent's name. `lym board open` then takes the run-agent refusal, and the scenario's `started: true` row fails.
   - The same holds for the `--board-path` rows unless the test empties the variable first.
   - Fix: pass `LYM_STRAND_NAME=` in `extraEnv` for every scenario call that should proceed, and state that the `--board-path` rows clear it with `t.Setenv`.

5. **MEDIUM — Card 5, D10/D11: the file-event wake of the watch loop has no row.**
   - `TestServer` wakes long polls by calling `BoardChanged` directly, and `TestRun` covers only the "`Watch` failing → timer" path.
   - No row injects a working `Watch` whose event on `board.json` wakes a waiting `GET /api/board`.
   - So D10's criterion "returns once another writer changes `board.json`" goes unreached on its main path, `internal/fswatch`.
   - Fix: add a `TestRun` row with an injected `Watch` whose channel delivers a `board.json` event after the file changes, asserting the waiter answers the new revision before any timer fires.

6. **LOW — Card 6, `TestBoardServeScenario` / PATTERN-test-speed: the second-`serve` row waits out a production duration.**
   - The row "`lym board serve` with an orch's strand name exits 0 answering `running`" runs a built binary, so it waits out the production five-second `LockWait`. PATTERN-test-speed says "No test waits out a production duration".
   - The behavior is already reached by `TestRun`'s `ErrRunning` row with an injected wait.
   - Fix: drop this step from the scenario.

7. **LOW — Card 4, `TestServer` CSP row: a wrong hash would pass.**
   - The row asserts only that the policy holds "a `sha256-` script source and no `'unsafe-inline'`".
   - A hash computed over the wrong bytes would block the page in a browser and still pass.
   - Fix: assert that the `script-src` and `style-src` hashes equal the base64 SHA-256 of the inline `<script>` and `<style>` content extracted from `ViewerPage()`.

8. **LOW — Card 3, `stepSyncCommitsViewerFiles` (tier1): repeats existing coverage.**
   - `Sync` stages the whole Bolt tree, which `stepSyncDirtyBoardCommitsAndPushes` already pins (`internal/boardengine#TestSync_Scenario`).
   - That both files are written is asserted at tier0 by `TestViewerPage` ("`RenderToDisk` writes both files").
   - The step can only fail when `RenderToDisk` omits a file, which the tier0 row already catches.
   - Fix: drop the step, letting the tier0 row plus the existing sync step carry D3's commit criterion.

9. **LOW — Cards 1 and 3: the manifest rows are duplicated and sit outside the existing manifest test.**
   - `TestRenderBoardData` asserts "the manifest lists `board-data.js`", and `TestViewerPage` asserts "`RenderToDisk` writes both files and its manifest lists both".
   - `internal/boardengine_test#TestRenderToDiskManifestCleanup` is the existing test of `RenderToDisk`'s outputs and manifest.
   - Fix: make it one assertion in `TestRenderToDiskManifestCleanup`'s seeded-manifest row, and drop it from both new tests.

10. **LOW — Card 6, `TestRunOpen`: the drain row repeats the leftover-state row.**
    - "Lock free while the old server's state is still on disk, as during a drain: a fresh spawn" has the same input as "lock free with a leftover state naming a port: one spawn".
    - Fix: drop one, or fold the drain wording into the other.

11. **LOW — Card 2, `TestExpectingRevision`: the no-`Expecting` row repeats existing tests.**
    - "The same edit through the board without `Expecting` is saved whatever the revision" restates the existing upsert tests (`internal/boardengine_test#TestUpsertTask`, `TestCLI`). The card itself says those keep passing unchanged.
    - Fix: drop the row.

**Tool calls beyond my context**

1. `internal/boardengine/runlock.go:205-242` and `internal/testkit/lymbin/lymbin.go:140-171`: to check whether `Title` is a run-locked scope field (it is, so card 4's run-lock row reaches) and whether lymbin passes on the inherited environment (it does).
2. `internal/vscode/keybindings_test.go` and the `Parallel` and hook installs in `internal/fabricengine/hubworkspace_integration_test.go` and the ide tests: to check whether a re-seed row already exists and whether the hook-install rows can run safely beside parallel subtests.
3. `internal/runproc/end_test.go:461-520`: to check whether card 6's `RoleBoardServer` record has a `HubOwner`-subject row to land in.
4. `internal/runproc/end_test.go:520-565`: to confirm the `HubOwner` row's expectations (`wantSignals`, `wantEnded`), which the new record changes.
