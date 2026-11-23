---
round: 1
focus:
  - "Fix item 2 of the board entry: does the diff make a run agent able to run one tmux-tier test through `lym gate test`, narrowly enough that it cannot start the whole tier? The diff only edits stencil prose; find where the refusal of tmux-tier runs for run agents lives and settle whether the diff changes it or leaves it, and whether the stencil text promises a call the gate still refuses."
  - "`tmuxScriptCommandPattern` and the `tmux-script` row in `internal/shellguard/rows.go`: does the pattern refuse the Seen command (`(script -qfc \"tmux ...\" /dev/null </dev/zero ... &)`) and a plain `tmux` in command position, and does it wrongly refuse commands run by agents in normal work, such as `lym reed`, `grep tmux`, `git commit -m` text, or a path like `tools/script`?"
  - "The new `SwitchAutonomous` wiring in `claudeengine/settings.go` and `codexengine/guards.go`: is each engine's installed and announced condition the same, and does every non-interactive launch, every role and fork, install the row while an interactive session keeps both commands? Check that the codex `autonomous` input is set at every `guardInputs` construction, not only in `Codex.Prepare`."
  - "The steer text `steerTmuxScriptDeny`: it names a form 'your instructions give for a single test'. Does every autonomous role's stencil give that form, so the way forward the refusal names exists for reviewers, drivers and fixers, and does the text follow `pattern/PATTERN-refusal-way-forward.md` (opens with the whole-command refusal, spells no forbidden form)?"
  - "Item 3 of the board entry: the writer and fixer stencil edits (`darn-fragment-make-change.md`, `darn-fragment-make-change-gated.md`, `gatefix-frame-cold.md`). Does each carry the rule that tool behaviour is learned from a repository test under the gate and that no background or detached process is started from Bash, and did any edit drop or weaken an existing rule, such as the ban on raw test commands or the one-class measuring call?"
  - "Commit trailers: the `Root-Cause:` trailer of a7937a620 against the defect the fix removes, and whether the changed test expectations in the engine tests and `settings-golden.json` loosen any assertion that existed at the review base."
---

Seeded from the board entry's Fix list and the diff's shape: one new shell guard row, one new switch in two engines, three stencil edits.
