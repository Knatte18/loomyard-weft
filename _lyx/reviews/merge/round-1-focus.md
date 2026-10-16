---
round: 1
focus:
  - "internal/webstercli/recoverbatch.go: the parent changed `reedUp` to `func() error` and removed the watch parameter, while the task added the `ErrRenameToolMissing` branch in the same verb. Settle whether the ordering of the reed boot, the spawn and the rename-tool refusal still holds, so `TestRenameToolMissingEnvelope` for recover-batch still exercises the refusal before any spawn or reed boot, and whether the fixture leaves `reedUp` nil or a stub that matches the new signature."
  - "internal/webstercli/cli.go and wiring.go: both sides added or changed fields and comments on `websterCLI` and `wireHub`/`wireStandalone` (`renameTools` against the `reedUp` signature). Settle whether the merged struct, the `wireStandalone` comment and the hub/standalone wiring statements are still true, in particular that `renameTools` stays empty in standalone and the removed watcher no longer matches any remaining prose."
  - "cmd/lyx/testdata/test-budget.yaml and internal/webstercli tests: the parent deleted tests from `wiring_test.go` and changed `cli_test.go`, and the task added a test and extended `TestWireHub_LanguageSkills`. Settle whether the `internal/webstercli` budget row still matches the merged top-level test count, and whether any merged test is redundant with another or any behavior of the interaction is uncovered."
  - "docs/overview.md and pattern/PATTERN-daemon-wakeups.md: the parent edited the overview and the daemon-wakeups list. Settle whether the task's code follows the changed wakeup rules, and whether the overview's webster and rename-tool statements still agree with the merged code."
  - "Callers of the removed watcher path: search the merged tree for remaining uses or descriptions of a standalone resize watcher, `reedEngine.Watch` or a two-argument `reedUp`, including `run.go` and the webster docs, which the task might still assume."
---

Seed focus from the overlap in `internal/webstercli`: the parent simplified the reed boot, and the task added the rename-tool refusal to the same verbs and wiring.
