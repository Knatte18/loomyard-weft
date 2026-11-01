# Round 1 fixer-report — plan

Every finding's premise held against the code; F1's premise was reproduced with a tmux 3.6 probe (`run-shell -c '#{@g}'` ran in the home directory, `cd` in the command reached `/usr`).

## Changes

- **F1 (BLOCKING), fixed differently in part.**
  - Card 1 adds `ChangeDirToWord(word)` per dialect (`cd <word>`, `Set-Location -LiteralPath <word>`), which takes an already-quoted word, with its row in `TestShell_ChangeDir`; the Intent, the header-comment count, the ImpactSummary and the commit title say four forms.
  - Card 9's button branch is `run-shell -b "#{E:@lym_button_command}"` with no `-c`. `composeButtonCommand` opens with `sh.ChangeDirToWord("#{@lym_button_dir}")`, and the card says why: `run-shell` format-expands its command but not `-c`. `toolfloor/doc.go` no longer gains `-c`, and the tier2 test's `pwd` assertion is named as the pin.
  - Card 8's confirm-before command is `run-shell -b <tmuxQuote(line)>`. `composeConfirmedButtonLine` takes `dir` and opens with `sh.ChangeDir(dir)`, every `#` doubled by `tmuxQuote`. `tmuxQuote`'s reason, the confirm test row and the quoting subtest follow.
  - Not made as proposed: the directory word `#{q:@lym_button_dir}`. The self-check showed that `q:` quotes POSIX-style, so under pwsh, which psmux runs `run-shell` lines in, a Windows path holding a space splits. Card 6 instead pins `@lym_button_dir` already quoted by `shell.ForGOOS().Quote`, and card 9 reads it plain, since tmux substitutes an option's value without expanding it again. A probe confirmed this: a value holding `#{session_name}` reached the shell literally. Card 6's `/b eam` row pins the quoted option value.
  - The decision-record request is deferred; see below.
- **F2 (MEDIUM), fixed as proposed.** Card 5 adds `internal/reedcli#TestSmokeSelvage` to its Edit group, with a Tests bullet saying `#{status}` reads `3` and its comment names the three-line bar; the ImpactSummary names it.
- **F3 (MEDIUM), fixed as proposed.** Card 4's live-pair row sets `parent_branch` to that pair's branch and the `gone-beam` row to a branch no worktree holds. A sentence says why: `ParentOf` answers the prime on a matching prime branch before it returns `parent_worktree`.
- **F4 (MEDIUM), fixed as proposed.** Card 6's `strand` is true only for an entry whose `run` or `confirm` still holds `{strand}` after the static fill. Its row splits into an agents entry with `{strand}` (`strand` true) and one without (`strand` false), both drawn only in the agent row. Card 8's step 3 is unchanged.
- **F5 (LOW), fixed as proposed.** In card 8 the gone-directory check is now step 5, ahead of the confirm step, now step 6. The removed-run-directory row covers a plain entry and an unconfirmed confirm entry, which issues no `confirm-before`.
- **F6 (LOW), fixed as proposed.** `TestRunButton_RealTmux` drops the changed-spec confirmed step and the no-name pane step.
- **F7 (LOW), fixed as proposed.** Card 8's bound paragraph adds that `--strand` lets a direct caller fill `{strand}` with any name `agentname.Parse` accepts, quoted per value, which adds nothing over running the command itself.

## Self-check

- fix-claims, MEDIUM: `ChangeDirToWord` over `#{q:}` is wrong under pwsh. Fixed with my own change: `@lym_button_dir` is pinned dialect-quoted and read plain (see F1). It merges with the fix-proposal MEDIUM below.
- fix-claims, LOW: card 4's reason said `ParentOf` checks the branch before it reads `parent_worktree`. Fixed as proposed: it now answers the prime before it returns `parent_worktree`.
- fix-proposal, MEDIUM: duplicate of the fix-claims MEDIUM. The fix-proposal lens suggested stating the degrade as accepted; my own change removes the degrade instead, as above.
- fix-consistency, LOW: the overview's card 1 index line lacked the directory change. Fixed as proposed.
- fix-consistency, LOW: the overview's Shared Decisions bullet lacked `ChangeDirToWord`. Fixed as proposed.
- fix-affected-set, LOW: `TestBindings_StatusBarClicksSelectAndZoomByRangeType`'s comment and `Fatalf` text still said two lines and `status 2`. Fixed as proposed: card 5's bullet names both.

Every lens returned a report.

## Disputed

None.

## Deferred

- **F1, the decision-record part.** The fix asks the parent to record that tmux 3.6 `run-shell` expands its command but not `-c`. This fixer's instructions forbid sending any message to another session about the review, and the decision record is outside the write surface. The loop owner should relay this. D7's click bullet (`run-shell -b -c '#{@lym_button_dir}'`) and its tmux-floor line (`run-shell -b -c`) should then read: the directory change rides in the pinned line, and the directory option holds the run directory quoted in reed's dialect. Card 9 states the departure and its reason in the meantime.
