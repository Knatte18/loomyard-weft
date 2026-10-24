```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out, building concrete cases where the proposed change is wrong.

**MEDIUM: the F4 proposal drops the Command entry's only no-dot reference row.**
- Changed text: card 20 Intent, "`TestCommandEntry` … deletes its `MissingSwitch`, `NonStringSwitch` and `ReferenceWithNoDot` rows."
- Why it's wrong: `ReferenceWithNoDot` is the only row of `TestCommandEntry` that reaches the dot check in `commandSettingRef` (`internal/shedrecipe/entries_command.go:57`). That check survives the card, because `command` stays a required `module.key` reference.
- Evidence: `entries_command_test.go:71` (`{"ReferenceWithNoDot", func(c Config) { c["switch"] = "open_window" }, …}`) is the only such row. The rows that remain for `command` are `MissingCommand` and `NonStringCommand`, and neither gives a no-dot value. The other no-dot rows (`entries_bouncer_test.go:338-340`) cover the Bouncer entry's own parser, not `commandSettingRef`.
- Case: after the card, a regression that accepts `command: "open_window_command"` passes every remaining row.
- Sound change: point the row at `command` (`c["command"] = "open_window_command"`, wanting `command` and the way forward) instead of deleting it. `MissingSwitch` and `NonStringSwitch` can still be deleted.

The other proposals hold up: F1's lists for cards 14, 20 and 13, F2, F3, F5 (`--gate <key>` exists from card 2 on), and the rest of F4 (the reopen row and the wire-test assertion).

Tool calls:
1. Grepped the shedrecipe tests and sources for no-dot and `module.key` coverage, because my context did not show whether another row covers `commandSettingRef`'s dot check.
2. Read `stepCommandRows_FollowTheTasksLocalSwitch` (`lifecycle_integration_test.go:1254-1282`), because my context had not shown that test's body, which I needed to judge the F4 marker-assertion proposal.
3. Grepped the plan for `--gate`, because my context did not show which card adds the flag that the F5 synopsis change names.
