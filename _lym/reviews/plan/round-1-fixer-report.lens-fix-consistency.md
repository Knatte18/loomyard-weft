```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: does the change set agree with its sibling cards, rows and cross-references?

1. **LOW**: `00-overview.md`, Card Index line for card 1, which reads "1 — shell-detached-forms — add output discard, zero exit and command flag to the pane-shell seam".
   - **Summary:** card 1's commit title now includes the quoted-word directory change, but the overview's index line for card 1 does not.
   - **Evidence:** card 1's `**Commit:**` now reads "Add output discard, zero exit, command flag and a quoted-word directory change to the pane-shell seam". The overview still lists only the original three forms.

2. **LOW**: `00-overview.md`, Shared Decisions, the bullet "Button rendering and the click verb use `shell.ForGOOS()` … and its new `DiscardOutput`, `ExitSuccess` and `CommandFlag` forms".
   - **Summary:** this bullet leaves out `ChangeDirToWord`, the fourth new `Shell` form card 1 now adds and card 9's click line uses.
   - **Evidence:** card 1's Create list adds `posixShell.ChangeDirToWord` and `pwshShell.ChangeDirToWord`. Card 9's `composeButtonCommand` is `sh.Chain(sh.ChangeDirToWord("#{q:@lym_button_dir}"), …)`.

Apart from these two lines, the edited text agrees with the cards around it:
- Card 8's reordered steps match its rows and card 9's note on a failed directory change.
- Card 8's `composeConfirmedButtonLine` signature matches its call site.
- Card 6's `strand` flag still holds against card 3's rule that `{strand}` needs `window: agents`.
- Card 5's new Edit target has its Tests bullet.
- Card 4's rows match `ParentOf`.

Out of my lens: D7 in `_lym/discussion/decision-record.md` and its tmux-floor line still spell `run-shell -b -c`. That depends on the decision-record request the fixer cannot make itself.

Further calls:
- Ran grep over `00-overview.md` and `geometry_integration_test.go`, because my context lacked the overview's text. The test-file path did not resolve from that directory.
- Read `00-overview.md` lines 12–30 to see the card index and the Shared Decisions bullets, which my context lacked.
