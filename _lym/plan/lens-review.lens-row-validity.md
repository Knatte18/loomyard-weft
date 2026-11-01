```yaml
lens: row-validity
outcome: reported
```

Findings (row-validity):

1. **MEDIUM** (card 14, D3 "renders the old entries or the new, never none"). The `TestRender` race row cannot reliably fail if the code is wrong. The goroutine removes `heddle.json` and writes a new one while holding the Bolt write lock, but nothing makes the render's read land inside that gap. A `read` that skips the lock would pass almost every run. Evidence: card 14 `TestRender` row "a render racing a goroutine that, holding the Bolt write lock, removes `heddle.json` and writes a new one". Fix: make the row deterministic. The test takes the Bolt write lock with `heddle.json` removed, starts `Render` in a goroutine, waits until the render is blocked on that lock, writes the new file, releases the lock, and asserts the new text.

2. **MEDIUM** (card 14, D3 "two renders of one worktree … leave one render's whole text"). The concurrent-render row cannot fail if the render lock is missing. Every write is already a whole-file rename of `.heddle/<file>.tmp`, so the final file is always one render's whole text with or without the lock. Evidence: card 14 `Render` bullet "writes each file by writing `<Root>/.heddle/<file>.tmp` … renaming it onto the target", and its row "two concurrent renders … leave a file equal to one render's whole text". The race the lock actually guards is both renders sharing one `<target>.tmp` path, so one rename can move half-written bytes. Fix: assert the lock itself. Hold `<ScratchDir>/heddle/render.lock` in the test, check that `Render` blocks until it is released, and that no `.tmp` exists while it is held.

3. **MEDIUM** (card 12, D7 "a run ledger set-aside with no run slug takes the run ledger's replayer tag"). The row "a ledger loss with an empty entry tags `set-aside/replayer/board/<short sha>`" has no input that reaches it. The board replayer always reports a non-empty `Entry`: a slug, `prime`, or a design file name. Evidence: `internal/boardengine#replayRun.lost` callers, `changePrimeOwner` reports `"prime"`, `applyDesignFile` reports `file.Name`. A fake run-ledger replayer named `board` that returns a loss with `Entry: ""` is needed, and card 12 does not list it beside the `fake` replayer.

4. **MEDIUM** (card 12 / card 16, `TestRecordsReplay` Bolt group). Process-wide registry restores race with each other across parallel rows:
   - The `scribbleThenFailReplay` row is a parallel subtest that registers its fake under the name `board` and restores "the previous set" afterwards (card 12).
   - Card 16's `registerHeddleReplayer` is a `sync.Once` that is never restored. If it fires in another parallel row while the scribble row's registration is live, the scribble restore puts back a set without `heddle`, and the Once never runs again.
   - Later heddle rows would then find `heddle.json` unclaimed and cherry-pick instead of replaying by entry. They would fail intermittently.
   
   Evidence: `internal/fabricengine/replay_integration_test.go:684-693` (parallel row with `t.Cleanup(InstallBoardReplay(...))`), and card 12's `RegisterBoltReplayer` "returns a function that reinstates the previous set". Fix: the restore removes or replaces only its own name, or the heddle registration happens before any row runs, e.g. in `newRecordsFixture`.

5. **LOW** (card 18, test economy). `TestHeddleCLI`'s refusal rows repeat `TestStore`'s rows (card 13): invalid slug, empty text, unknown target, missing flags on a new entry, unknown slug on remove. Only the CLI-specific part is new behaviour: the message is printed, nothing is saved and no sync is spawned. One refusal row for `set` and one for `get`/`remove` would cover that. Evidence: card 13 `TestStore` rows "`Set` refuses a bad slug, an empty text, an unknown target and a new entry missing its order or targets"; card 18 row "`set` refuses an invalid slug, an empty text, an unknown target and a new entry missing flags".

6. **LOW** (card 18, nil seams). `heddleCLI` gains `save` and `spawnSync` fields, and its `Renderer`'s wait must be 50 ms for the lock-wait row, so the test builds the receiver by hand through `commandOf`. The card states only the defaults `Command()` fills, not what a nil `save` or `spawnSync` means on a hand-built receiver. Evidence: card 18 "`heddleCLI` holds … two seams a test replaces: `save` (default `heddleengine.Save`) and `spawnSync` (default `boardengine.SpawnSync`)".

7. **LOW** (card 9, `TestWorktreeSeam_Events`). The row "with no seam installed, the same pull records nothing" is vacuous: with no seam installed the fake is not called, so "records nothing" cannot fail. Only the "returns no warning" half asserts anything. Evidence: card 9 last bullet.

8. **LOW** (card 17, D4 "a resume through the launch's own resume line keeps the file"). The row only checks that `Launch.ResumeCmd` names the same file. `TestPrepare_AppendSystemPromptMatchesDenyNotice` already asserts that for both lines, so this half adds no coverage. Evidence: `internal/shuttleengine/claudeengine/prepare_test.go:316-321`.

Tool calls:
- One grep combining `internal/hubforge/hub.go` (checking that `NewHub` goes through `CloneAndWire`, card 9's fixture claim), `_board` occurrences in `cmd/lym/stencilseed_integration_test.go`, and `scribbleThenFailReplay` at `replay_integration_test.go:60-80` for its delegation and install shape. My context lacked all three.
