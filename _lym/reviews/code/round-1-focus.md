---
round: 1
focus:
  - "Board entry's second hold: `internal/battenshed/innerrun_pause.go` holds a covered run's spawn, and `lym s` reached it. The diff leaves that file untouched. Settle whether `lym s` (adopt --here, beam start, reed attach) now passes a run-scoped pause, or whether the entry's `lym s` case is unmet."
  - "Board entry's bullets: a test that a paused hub does not hold `board get`, `board list`, `machine name`, `selfreport create` and `lym s`, and a held line printed at once by a verb that does wait. Settle which of these the diff delivers; `cmd/lym/main_test.go` and `internal/tendercli/cli_test.go` are the places to look."
  - "`heldByPause` in `internal/tendercli/gate.go`: does the run-process mark reach every process the entry means to hold (strand agents, the loop and its children, push children), and does any operator verb started from an agent or run process wrongly inherit `LYM_RUN_PROCESS`? Check the unparsable-strand-name and orch-strand branches."
  - "Annotation coverage: only shed `run`/`step` and `beam dobby` carry `RunProcessAnnotation`. Settle whether other run-loop entry points that a pause is meant to stop are marked or reached through inheritance, and whether `ExemptAll` for an unmarked process drops a hold that `DetachedChild` previously gave."
  - "Env scrub copies: `stripIdentityEnv` in `internal/reedengine/lifecycle.go` and `ServerEnviron` in `internal/testkit/tmuxkit/tmuxkit.go` (a string literal, not `pauserecord.RunProcessEnv`). Settle whether every other path that spawns a tmux server or pane (reed engines, shuttle engines) strips the mark, and whether the literal can drift."
  - "`InstallNetworkGate` calls `os.Setenv` before the hub check; settle whether marking the environment when no gate is installed (outside a hub) or in tests leaks state, and whether the signature change reached every caller."
  - "Existing callers of `SetGateExemptAll` (`lym adopt`, fabric teardown, `lym batten cancel`): settle whether they are still needed and whether the changed doc comments in `internal/pauserecord/doc.go` and `hold.go` match the behavior."
  - "Test economy: check `TestStripIdentityEnv_DropsNameParentAndRunMarkOnly`, `TestEnvScrub_MatchesTmuxkitServerEnviron`, `TestRunCommand_MarksOnlyARunsOwnVerbs` and `TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub` against the redundancy report and their covering tests."
---

The answer key is the decision record's board entry under `## Goal` (issues #129 and #124); its other sections are empty.
