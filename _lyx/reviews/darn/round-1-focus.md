---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/command.go: does the empty-mode default resolve to bypass for every caller of `validatePermissionMode` (autonomous, interactive, opened), keep `prompt` refused on an autonomous run, and leave the launch and resume lines consistent?"
  - "internal/shuttlecli/open.go: does the success envelope report the engine, model and effort the session actually got (empty meaning provider default), and does `--list-models` open nothing, need no `--role`, and sort by alias with engine, model and default effort?"
  - "Board entry scope: does the diff cover each ask (bypass default per engine, discoverable model-spec grammar in `shuttle open --help`, shown choice, list of choices, `reed add` help pointing to `shuttle open`) and make no change the entry does not license?"
  - "internal/reedcli/add.go: is the broken `%SID%` example gone and does the help send an agent launch to `lyx shuttle open`?"
  - "Docs in the same diff: do the `Spec`/`OpenSpec` comments, `internal/shuttleengine/doc.go` and `docs/overview.md` match the new behavior, with no stale mention of an interactive prompt default?"
  - "Test economy: do the new and changed tests in cli_test.go, command_test.go and prepare_test.go duplicate existing coverage, and is any added behavior uncovered?"
  - "Change description: check each claim in `_lyx/landing/summary.md` against the diff, notably the claim that every shuttle `PreToolUse` guard stays on and that interactive runs now bypass."
---

Seed focus drawn from the board entry and the diff stat; nothing here pre-states a verdict.
