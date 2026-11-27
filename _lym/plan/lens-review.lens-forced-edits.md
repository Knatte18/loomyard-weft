```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens: what each card's change forces outside the members it names.

- **BLOCKING — Card 1, the new refusal row of `TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake`.** The shared test helper `internal/dobbyengine#lookupAsText` sends every key it does not recognise to `LookupPullRequest`, and no card lists the helper.
  - Evidence: at `internal/dobbyengine/tasksetting_test.go:31-53`, the helper branches only on `LandNoticeKey`, `AutonomyKey` and `RunWatchKey`. `TestLookupTaskSettings` calls it at `:124`.
  - Unconfirmed: I did not read past `:280`, so I have not seen that the refusal test itself calls the helper.
  - If it does, a row keyed `window_placement` reads `pull_request` instead. That key is unset, so the template's `review` comes back with no error and the row fails.
  - Fix: add `internal/dobbyengine#lookupAsText` under Card 1's `**Edit:**`, with a `WindowPlacementKey` branch that calls `LookupWindowPlacement`.

- **LOW — Card 1, the `tasksetting.go` file comment.** The Intent says it rewrites this comment, which lists the four per-task keys. The comment belongs to no listed member, and the card names no target for the file.
  - Evidence: `internal/dobbyengine/tasksetting.go:1`.
  - Fix: list `internal/dobbyengine/tasksetting.go#`, or a member glyph whose span holds the comment.

- **LOW — Card 3, the `timing` struct's new field.** `internal/desktophook#AfterWindowWithTiming` builds `timing{sleep, commandTimeout, commandWaitDelay}` by field name, so `groupKillAfter` stays zero there.
  - Evidence: `internal/desktophook/export_test.go:14`.
  - `TestAfterWindow`'s `HangingCommandIsCutOffAtTheTimeout` row then runs under a group kill whose zero-delay meaning no card states. No card lists the helper.
  - Fix: list `internal/desktophook#AfterWindowWithTiming` under `**Edit:**`, and either state what a zero `groupKillAfter` means or pass a value.

- **LOW — Card 11, `internal/reedcli/doc.go` under `**Extends:**`.** Naming the new verb rewrites the existing sentence "`window`, `follow` and `button` are run by launchers and tmux bindings, not by hand." It does not only add to it.
  - Evidence: `internal/reedcli/doc.go:25`, and the rubric's "Extends on a ref the card rewrites".
  - Fix: move `internal/reedcli/doc.go` under `**Edit:**`.

- **LOW — Card 11, the file comment of `internal/reedcli/cli.go`.** The comment names the verbs that skip the pre-run: "The watchdog, switch, follow, button and window verbs skip that resolution". Card 11 adds `place-window` to that skip set.
  - The comment sits outside `internal/reedcli#Command`'s span, which is the card's only target in the file.
  - Evidence: `internal/reedcli/cli.go:1-3`.
  - Fix: list `internal/reedcli/cli.go#` or a member glyph that covers the comment.

- **LOW — Card 13, the overview doc.** D12's bound says that under placement, `lym machine start`'s windows gain the `window_command` they never ran. `docs/overview.md` documents `lym machine start`'s windows, but only Card 12 lists that file, and only for the `lym reed window` paragraph.
  - Evidence: `docs/overview.md:911`, and the record's D12 bound.
  - Fix: Card 13 lists `//docs/overview.md` and says the started windows run the window command under placement.

- **LOW — Shared Decision `contract-test-tier` (Card 7).** Putting a real-desktop test in `tier3` makes two pattern background files wrong, and no card edits either of them.
  - `pattern/PATTERN-test-speed.md` defines `tier3` as "a real LLM session" in its four-tier table.
  - `pattern/PATTERN-test-stand-ins.md` says a contract test lives in "the tier that needs the real thing".
  - Evidence: `pattern/PATTERN-test-speed.md:15`.
  - Fix: Card 7 extends `pattern/PATTERN-test-speed.md`'s tier table so `tier3` also covers a real desktop session, the operator's call.

Every other glyph the plan lists resolves, including `internal/boardcli#storeVerbs`, `internal/proc#ConfigureGroupTerminate`, `internal/desktophook_test#newDirsWithMachineConfig`, `internal/runproc#Register`/`ChildRecord`, `internal/fabricengine#PrimeName`/`ConfigDirs`/`List` and `internal/lymcwd#NamedWorktree`.

Tool calls:
1. `lym quarry resolve` on the glyphs the cards list under Uses and Edit, which my context had not resolved.
2. A grep for the users of the `openTerminal` and `terminal` seams, the callers of `runWindowCommand` and `afterWindow`, and the signature of `ConfigureGroupTerminate`, to find callers the re-signs force.
3. A grep for runproc role lists, the `lookupAsText` helper in `internal/dobbyengine/tasksetting_test.go` and other readers of the per-task keys, to find registrations and helpers the new key and role force.
4. A read of `TestLookupTaskSettings`' use of the helper, `internal/runproc/doc.go`'s role text, `internal/runproc/end_test.go`'s role list, `internal/reedcli/doc.go`'s verb list and the runproc spare lists, because my context had not read those spans.
5. A grep of `internal/lymdirs/doc.go`, `pattern/PATTERN-sole-parsers.md`'s format list, the machine-start text in `docs/overview.md`, the refusal test's rows and any help-tree goldens, to find forced doc and registry edits.
