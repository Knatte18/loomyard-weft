```yaml
lens: code-affected-set
outcome: reported
```

**Lens list: code-affected-set. Everything in the set I found is covered, and I have no findings.**

1. **`Config.StrandName` and the `boardengine.Board` field.**
   - Config constructions: `boardengine.New` is called from `internal/boardcli/cli.go:147`, `internal/battencli/wire.go:1211` and `:1238`, and `internal/ideengine/menu.go:38`. Only boardcli sets the strand name, which matches the stated bound and the "Open risks" entry.
   - The `battencli` calls build a bare `Config` with an empty strand name. That is the operator path, so it stays unguarded. Covered.
   - The `ideengine` call is a read menu. Covered.

2. **`checkTaskCreation` in `boardCriticalSection`.**
   - Write paths: every write method goes through `boardCriticalSection`, so promote, upsert, batch, merge and rename are all reached. Covered.
   - Detached sync: it uses the `--board-path` config and then `cfg.StrandName = strandNameFromEnv()` (`cli.go:145`). The sync child inherits the parent's env and only syncs, so it is harmless.
   - Intake: `intake import` runs `b.ImportIssue` (`intake.go:296`) on the same pre-run board, so it is guarded too. I did not check whether an import can yield `kind` task; I assume it files notes.
   - Local-commit replay: it does not go through the critical section, and `boardengine/doc.go` says so. Covered.

3. **The boardcli pass-through (`cli.go:145`).**
   - Every verb's board comes from the one `PersistentPreRunE`. The only early return, for the `board` and `intake` parent commands, runs no write. Covered.
   - The existing `prepareRunSettings` calls (`cli.go:519`, `:585`, `:903`) still read the env themselves through the same `strandNameFromEnv`. This is consistent, and the engine never reads the env.

4. **Docs.**
   - `boardengine/doc.go`, `boardcli/doc.go` and the board `SKILL.md` all state the refusal and the orch exemption, and they match the code.
   - The `SKILL.md` text names `orch` and `orch-2`, which matches `MatchesRole`.
   - The parent's merged changes to `cli.go`, `doc.go`, `verbs_test.go`, `boardengine/doc.go` and `SKILL.md` touch the profile text only. They are orthogonal and do not conflict with the guard text. Covered.

5. **Stencils.**
   - The follow-up-as-note sentence is added to `darn-template-write.md`, `loom-template-discussion.md` and `loom-template-plan.md`, beside the existing own-entry rule.
   - `loom-rubric-discussion-review.md` and `loom-rubric-plan-review.md` each gain a flag bullet.
   - All additions are purely additive, so no existing rule is dropped or weakened.
   - The orch stencil is untouched, which is right now that D5 is withdrawn.
   - `contracts/stencils/loom/loom-rubric-code-review.md` also carries the own-entry "board lock refuses" text and is not extended. The decision record's scope lists only the discussion and plan rubrics, so this is not a finding.

6. **Tests.**
   - `taskguard_test.go` covers the engine paths.
   - `cli_test.go` and `verbs_test.go` register `stepTaskSessionFilesNotesOnly` for the CLI pass-through. Covered.

Tool calls:
1. `grep` for "board lock refuses" and for `boardengine.New(` callers: to find sibling stencils and other `Config` constructions, which my context lacked.
2. `grep` in `boardcli` for `boardengine.New`, `StrandName`, `PersistentPreRun` and `Promote`: to confirm a single wiring point.
3. `sed -n` on `cli.go:104-150` and `intake.go:285-300`: to check for early returns before line 145 and the intake import path.
