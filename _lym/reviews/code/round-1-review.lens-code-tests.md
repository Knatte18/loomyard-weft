```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests. For each behaviour the diff adds or changes, I name the test that pins it, or "none".

**Behaviour → pinning test**

1. Set renders every file target into the prime and a task pair, and prints `rendered` and an empty `failed`: `TestHeddleCLI/WriteRendersEveryLiveWorktree`, first `set` (`cli_integration_test.go:546-585`). Pinned.
2. `failed` prints as `[]` and not null on success: the `failures` helper in the same subtest fatals if the key is missing, and the first `set` asserts it is empty. Pinned.
3. A prunable worktree (directory removed) is skipped and is not reported as failed: same subtest, last step (`:617-626`). Pinned.
4. A worktree left `Unwritten` by an unmarked file lands in `failed` with the worktree name and the file name, and the verb still succeeds: same subtest, the `remove` step. Pinned.
5. Remove renders the prime: the same `remove` step. Pinned.
6. `--no-render` on `set` omits `rendered` and leaves the earlier render untouched: same subtest, middle step. Pinned.
7. `--no-render` on `remove`: none.
8. `--no-render` on `set --layer lym` and `remove --layer lym`: none. The flag is wired separately at `verbs.go:217`, `:269` and `:303`.
9. The render after an override-layer set or remove: none. `OverrideVerbs` (`:317-360`) runs those writes but asserts nothing about `rendered` or `failed`.
10. `rendered` prints as `[]` when nothing rendered (the `nonNil` path): none. The only run with no rendered worktree is the `--layer` case, which has no assertion.
11. A `git worktree list` failure gives the single `render_warnings` entry with no render: none.
12. `render_warnings` carries the warnings of successfully rendered worktrees, prefixed with the worktree name: none. The unmarked-file warning goes to `failed` instead, so the `default:` branch with warnings is never reached.
13. The error branch of the switch (`NamedWorktree` fails, `EventFor`/seed fails, or `Render` errors, such as the Bolt lock held) gives a `failed` entry with the `lym heddle render <name>` way forward: none.
14. An `Unwritten` result with an empty `Warnings` list gives a usable cause: none. The test case always carries a warning, so the text after "left unwritten:" is never checked empty.
15. A worktree with a non-`.` anchor (`Shape{Anchor: "backend"}`) is rendered at its own anchor by the fan-out: none. Only the `render` verb is tested with an anchor (`RenderWritesFileTargetsAtTheAnchor`).
16. A beam worktree: none, but a beam is a pair to this code, so I judge it covered by the pair case.
17. The sync starts after the renders and no render waits on the Bolt lock: none. `WriteHoldsTheBoltLockAndCommitsNothing` (`:456`) pins the lock being free during the sync. It does not pin the order relative to the renders, so moving `syncAfterSave` before `renderEveryWorktree` would still pass. The commit message and `doc.go` claim this order.
18. The sync still starts and its warning is printed when the render is on: `SyncFailureKeepsTheSave` (`:396`) runs with the default render. Pinned.
19. A refused change renders nothing: none. `RefusalsSaveAndSyncNothing` and `EmptyTargetsOnExistingEntryIsRefused` assert saves, syncs and the store file, not worktree files.
20. `ChangeStore`/`changeStore` render no worktree and keep returning the sync warning: the warning and errors are pinned by `ChangeStoreReturnsErrorsAndWarning` (`:414`). "Renders no worktree" is not asserted.
21. `renderInto` extraction keeps the `render` verb behaviour: the happy and unmarked paths are pinned by `RenderSeedsAndWritesThePrime`, `RenderWritesFileTargetsAtTheAnchor` and `RenderWarnsOfAnUnmarkedFile`. The Bolt-lock refusal and its `lym heddle render` text are pinned by `RenderRefusesWhileTheBoltLockIsHeld`. The seed-failure message changed to "seed the exclude lines: …": none.

**Findings**

- **MEDIUM, design (coverage gap).** Items 7-9: `--no-render` on `remove` and on both `--layer lym` write paths has no test, and the override-layer render has none either. Evidence: `verbs.go:217`, `:269`, `:303`, `:305` and `cli_integration_test.go:317-360`, where `OverrideVerbs` asserts no render output. Fix: add rows or assertions to `WriteRendersEveryLiveWorktree` or `OverrideVerbs`. Do not add a new test.
- **MEDIUM, design (coverage gap).** Items 11 and 13: the list-failure warning and the per-worktree error branch (`failed` with the way forward) have no test. Evidence: `verbs.go:454-457` and `:469-470`. The Bolt lock held, with `BoltLockWait` shortened as in `RenderRefusesWhileTheBoltLockIsHeld`, would drive the error branch, and the same subtest could carry it.
- **LOW, design (coverage gap).** Item 12: no test reaches the `render_warnings` output, the field the diff adds. Evidence: `verbs.go:424-426` and `:475-477`.
- **LOW, design (coverage gap).** Item 17: nothing pins the render-before-sync order the commit message and `doc.go` state. Evidence: `cli_integration_test.go:480-486` checks the lock only. Observing at `spawnSync` time that the pair's `CLAUDE.local.md` already exists would pin it.
- **LOW, design (coverage gap).** Items 15 and 19: fan-out at a non-`.` anchor, and a refused change leaving worktree files untouched, have no test. Evidence: `:362-394` assert only saves, syncs and the store file.
- **NIT, consistency.** The comment at `:504` ("no worktree file changed") and the assertion at `:509` now hold only because the rendered files are git-excluded. They do not prove that nothing was rendered. Evidence: `:509-511`.
- **Test economy:** I found no duplicate test. The new subtest covers a behaviour no other test covers. `TestHeddleCLI` is not judged by the redundancy report (unclassifiable), so no `//testtiming:keep` finding is warranted.

**Tool calls:** none. My context already held every file I needed.
