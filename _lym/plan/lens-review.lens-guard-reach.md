```yaml
lens: guard-reach
outcome: reported
```

Guard-reach findings for the board-viewer plan. Every guard was traced from its real entry point. Findings 1, 2, 3 and 6 also call for a change to a card; findings 4 and 5 only name rows the plan lacks.

1. **BLOCKING — card 4, `applyWrite` (D10 create): every create of a new slug is refused as stale.**
   - Card 4's Intent says `applyWrite` first reads the entry from `Board.View()`, and that "a missing entry … answers the stale refusal from that view without a write". That pre-check is applied to every op.
   - A create names a slug the board does not hold yet, so its entry is always missing at that point, and the write never runs.
   - The `Expecting(slug, "")` create path the card describes cannot be reached for a new slug.
   - The row "ops table: create … through `POST /api/write` … answer the fresh entry" fails as written.
   - Fix: exempt `create` from the missing-entry branch. A create should go straight to `Expecting(slug, "")`. Its pre-check, if any, should refuse only a slug that exists, answering with that entry.

2. **MEDIUM — card 4, `draft` and `undraft` guards (D10: "draft a task whose status is none; undraft a task whose status is draft"): the guards check status but not kind.**
   - As written, `draft` is refused only when the status is non-empty, and `undraft` only when the status is not `draft`. Neither checks that the entry is a task.
   - `Store.SetStatus` → `validateWrite` (`internal/boardengine/store.go:376-470`) puts no rule on a note's or project's status. So `draft` on a note or project with no status writes `status: "draft"` to it.
   - `undraft` on a project in `draft` calls `SetStatusReleasingProject(slug, nil)`. That also releases every draft member task (`internal/boardengine/projectrelease.go:31-56`), which is a write D10 does not license.
   - Release and promote do check kind; draft and undraft do not.
   - Fix: refuse a non-task for both ops, and add those rows to `TestServer`.

3. **MEDIUM — card 6, `TestServeOpenRefusals` (D12 criterion): the run-agent row is preempted unless it passes `--board-path` or has a hub cwd.**
   - The run-agent refusal sits in `RunE`. The board subtree's `PersistentPreRunE` runs first, and without `--board-path` it calls `lymcwd.Resolve(cwd)` and aborts on a non-hub cwd (`internal/boardcli/cli.go:139-150`).
   - A tier0 test cannot use `seedCwd`, because it runs `git init`.
   - So the row reaches the refusal only when the invocation also carries `--board-path`. The run-agent check runs before the `--board-path` refusal, so that combination reaches it.
   - The card states neither. Fix: say the row passes `--board-path` together with the strand name.

4. **LOW — card 4, `release` of a non-task:** the branch "any other non-task is refused naming `lym board set-status`" is reachable through the API with a note's slug, but `TestServer` plans no row for it.

5. **LOW — card 6, the opener wait's strand cap:**
   - The cap applies only to a strand name that passes `refuseRunAgent`: an orch's name, or a slug-free name.
   - `TestRunOpen`'s row "with a strand name, an injected cap and an opener that never exits: success at the cap" reaches it only if that strand is an orch or slug-free name.
   - With a run agent's name, `refuseRunAgent` refuses first and the cap is never reached. The row should name an orch strand.

6. **LOW — card 8, `regenerateHubWorkspace` members for a beam: an error branch is left unstated.**
   - The beam anchor comes from `lymcwd.NamedWorktree`, but the card does not say what happens when that call fails.
   - The same call already fails softly in `beamWorkspaceSettings`, which warns and falls back (`internal/fabricengine/hubworkspace.go:101-110`, and its caller at :198-201).
   - A new hard failure here would turn a degraded regeneration into an error that writes nothing. The card should name the fallback, for example skipping the hook's members or using the beam's folder path.

7. **LOW — cards 6 and 4, tests that depend on `LYM_STRAND_NAME`: an ambient value can preempt them.**
   - `serve` and `open` refuse a run agent's strand name before they do anything else, and `checkTaskCreation` reads the same variable.
   - `TestServeOpenRefusals`' `--board-path` row reaches its refusal only if the run-agent check passes. `TestBoardServeScenario`'s built `lym` inherits the environment.
   - A gate run started from a run agent's session could carry that agent's strand name. The rows should set `LYM_STRAND_NAME` to `""`, or to an orch name, explicitly; existing tests set it explicitly (`internal/boardcli/verbs_test.go:162,1241`).
   - Card 4's server builds its board with `StrandName` emptied, so it is not exposed.

Tool calls I made:
- Read `internal/boardengine/runlock.go:150-242` (`checkRunLocks`, `scopeChanged`), to confirm that a title edit counts as a scope change, so card 4's run-lock row is reachable. It is; my context had only the function names.
- Read the status and kind lines of `internal/boardengine/store.go:376-470` (`validateWrite`), and grepped for strand-env setup in the test helpers. This checked whether a note can take a status, and how tests set `LYM_STRAND_NAME`; my context lacked that function's body and the test env handling.
