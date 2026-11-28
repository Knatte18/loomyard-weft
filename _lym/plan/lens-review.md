# Lens review

Every lens returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Duplicates across lenses are merged into one line, which names each lens and finding it covers.

- way-forward F1, guard-reach 1, row-validity 1, coverage 8 — BLOCKING — fixed differently: the proposals gate a create on the view's slug absence alone, which card 4 now does, but coverage 9 showed that the view pre-check also hid the write section's stale refusal for every op; card 4 now pre-checks only release, draft and undraft against the view (their rule reads kind and status), sends edit and promote straight to `Expecting`, refuses a create of a slug the view holds with that entry, and gains a create-of-an-existing-slug 409 row; the overview's Shared Decisions bullet says so.
- coverage 9 — MEDIUM — fixed differently: the proposal adds a hook between the pre-check and the write; dropping the edit's pre-check (above) makes the existing stale-edit row reach the write section's `*StaleEntryError` → 409 mapping with no hook, and a stale release row now reaches the view's refusal.
- way-forward F2 — MEDIUM — fixed as proposed: card 3 states the form's held revision becomes the fresh entry's on a stale refusal and the answered entry's on success, and what a null fresh entry does.
- way-forward F3 — LOW — fixed as proposed: card 4's undraft refusal splits by status, naming that an empty status needs nothing, `lym board set-status` for a run status and the page's release for any other.
- way-forward F4 — MEDIUM — fixed as proposed: card 5's `Run` truncates the log once it holds the lock and writes its pid as the first line, so the log of the lock holder opens with its pid; card 6's bound refusal states it.
- way-forward F5, coverage 2 — MEDIUM — fixed as proposed: card 6's opener failure names the launcher path to open in a browser by hand, and its `TestRunOpen` row asserts it.
- way-forward F6, coverage 1 — LOW/MEDIUM — fixed as proposed: card 6's `openDeps` gains a `Prime` seam and `TestRunOpen` a failing-`Prime` row asserting the error and no spawn; the error stays the one `lymcwd.NamedWorktree` gives, as D5 states.
- way-forward F7, coverage 3 — LOW/MEDIUM — fixed as proposed: card 6 returns a spawn error naming the server log and its way forward, with a failing-`Spawn` row.
- guard-reach 2 — MEDIUM — fixed as proposed: card 4's release, draft and undraft refuse a non-task, with draft, undraft and release rows for a note.
- guard-reach 3, coverage 20 — MEDIUM/LOW — fixed as proposed: card 6's `TestServeOpenRefusals` runs both rows with `--board-path`, and states the run-agent check comes first.
- guard-reach 4, coverage 10 — LOW/MEDIUM — fixed as proposed: card 4's `TestServer` gains a release-of-a-note row.
- guard-reach 5, row-validity 3 — LOW/MEDIUM — fixed differently: the proposals name an orch strand in the cap row or state the zero cap; card 6 lists `openDeps`' fields, makes a zero `OpenerCap` mean no cap with the verb setting it only for a non-empty strand name, and the rows drive the cap directly (non-zero cap at the cap, zero cap waiting for exit), so the strand choice stays in the verb's one line.
- guard-reach 6 — LOW — fixed as proposed: card 8 logs a beam whose anchor cannot be read at Warn and builds its file with no hook members, as `beamWorkspaceSettings` degrades.
- guard-reach 7, row-validity 4 — LOW/MEDIUM — fixed as proposed: `TestServeOpenRefusals` sets `LYM_STRAND_NAME` to `""` with `t.Setenv` for its `--board-path` row, and `TestBoardServeScenario` passes `LYM_STRAND_NAME=` in every call's extra environment.
- forced-edits 1 — BLOCKING — fixed as proposed: card 6 lists `internal/boardengine#Board.UpsertTaskWithRecord`, the second caller of the deleted `isRunAgentSession` (`internal/boardengine/recordupsert.go:29`).
- forced-edits 2 — BLOCKING — fixed as proposed: card 1 lists `internal/boardengine_test#TestOutputs` (`internal/boardengine/config_test.go:345-380`), whose rows want the new `Labels`.
- forced-edits 3 — BLOCKING — fixed as proposed: card 4 creates `internal/boardserve/testmain_test.go`, untagged, calling `gitkit.HermeticGitEnv()` and `tmuxkit.Main` (`pattern/PATTERN-test-isolation.md:7,13-17`).
- forced-edits 4 — MEDIUM — fixed as proposed: card 7 edits `docs/overview.md`'s `tasks.json` and key-block sentences (`docs/overview.md:582,606-608`), and card 9 extends its workspace-file passage with `beam start`.
- forced-edits 5 — LOW — fixed differently: card 6 rewords `rerender`'s `Short` (`internal/boardcli/cli.go:406`); the file comment at `internal/boardcli/cli.go:3-5` is not changed, since it lists the verbs Command builds with "such as", so `open` and `serve` leave it true.
- forced-edits 6 — LOW — fixed as proposed: card 1's escaping bullet names the JSON unicode escapes `003c`, `003e` and `0026` as D1 words them.
- coverage 4 — MEDIUM — fixed as proposed: card 8's refusal row's hook refuses only for a beam's file, and the row asserts the hub file is unchanged too.
- coverage 5 — MEDIUM — fixed as proposed: card 9 adds tier0 `TestWorkspaceTasks` (empty `lymPath` gives `lym`) and `TestWorkspaceContent` (root mapping, beam slug, a failing `LymBinary` giving `lym`).
- coverage 6 — MEDIUM — fixed differently: the proposal asserts the git state, which a tier0 `SkipGit` board has none of; card 2's row now snapshots every file of the board directory, manifest included, and asserts no change and no added file, which is what a sync stages, since the refusal returns before the save, render and sync steps.
- coverage 7 — MEDIUM — fixed as proposed for card 4: `TestServer` sends a second edit carrying the revision the first write's answer returned and asserts it is saved; card 2's row stays, since the server's answer is built from the `Board.View` that row reads.
- coverage 11, row-validity 5 — MEDIUM — fixed as proposed: card 5's `TestRun` gains a working-`Watch` row whose event wakes a waiting long poll before any timer fires.
- coverage 12 — MEDIUM — fixed as proposed: card 5's in-flight row asserts `Live` reads false while the write is still blocked.
- coverage 13 — MEDIUM — fixed as proposed: card 5's idle row adds an `/api` request that defers the exit past the bound measured from the start.
- coverage 14 — MEDIUM — fixed as proposed: card 5's `TestRun` asserts `Register` was called once and the log's first line names the pid over a leftover log.
- coverage 15 — MEDIUM — fixed as proposed: card 5 states `Live` reads a missing state directory as no server, creating nothing, with a row.
- coverage 16 — MEDIUM — fixed as proposed: card 1's `TestRenderBoardData` asserts `sections` order, `types`, `areas`, `prime_owner`, `rendered`, `design_file`, `recipe`, `owner`, `created` and `revised`.
- coverage 17 — MEDIUM — fixed as proposed: card 1's board gains a done project with an open sub-project and a task, with rows on the sub-project's `P0` parent and the task's project handle.
- coverage 18 — LOW — fixed differently: the proposal compares against the CLI verb itself, which needs a tier1 git hub for `runCLI`; card 4's draft now makes the CLI's own `SetStatusReleasingProject` call, the twin row names the call the CLI's RunE makes, and the card's bound states that the CLI's other upsert steps act only on fields no page op accepts.
- coverage 19, row-validity 2 — LOW/MEDIUM — fixed as proposed: card 6's unreachable run-agent row is dropped from `TestRunOpen`; `TestServeOpenRefusals` carries it.
- coverage 21 — LOW — disputed, every fix worse: `serve` refuses a run agent's strand before it builds its board (card 6), and `isRunAgentSession` is false for an orch or slug-free name (`internal/boardengine/taskguard.go:14-17`), so no strand name that reaches `serve` changes a write's outcome; a row could not tell an emptied strand from a passed one.
- coverage 22 — LOW — fixed as proposed: card 5 asserts a 32-hex-character token, and card 4's CSP row checks both hashes (row-validity 7).
- coverage 23 — LOW — fixed as proposed: card 5's leftover-state row reads a state holding an unknown key through `ReadState`.
- row-validity 6 — LOW — fixed as proposed: the second-`serve` step is dropped from `TestBoardServeScenario`; `TestRun`'s `ErrRunning` row covers it with an injected wait.
- row-validity 7 — LOW — fixed as proposed: card 4's CSP row asserts each hash equals the SHA-256 of the inline element the test extracts.
- row-validity 8 — LOW — fixed as proposed: `stepSyncCommitsViewerFiles` and card 3's `TestSync_Scenario` edit are dropped; card 3 states the existing dirty-board step pins that the sync stages the whole tree.
- row-validity 9 — LOW — fixed as proposed: the manifest assertions move into `TestRenderToDiskManifestCleanup`'s seeded-manifest row, `board-data.js` on card 1 and `board.html` on card 3.
- row-validity 10 — LOW — fixed as proposed: card 6 folds the drain row into the leftover-state row.
- row-validity 11 — LOW — fixed as proposed: card 2's no-`Expecting` row is dropped; the existing CLI write tests pin it.
