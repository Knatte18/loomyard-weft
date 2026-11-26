---
round: 1
focus:
  - "internal/reedengine/clientrow.go `readClientFormatsVia`: check how `TmuxCmd.output` trims its result, and whether a row whose trailing format is empty (an unset pane option, last client listed or not) still splits into the expected field count. Settle what each of the three callers then does and reports, and whether the tests cover that shape."
  - "internal/reedengine: check whether any other read that must follow the clicking client still goes through `display-message -c` without `-t`, or through another call that tmux expands against the most recently active client, on the `reed switch` and button-click paths (confirmed and answered clicks included)."
  - "internal/shellguard/rows.go `sessionBusCommandPattern`: compare what it refuses and lets through with the amended entry's list, including `dbus-send` with and without `--session` or `--system`, and the spellings of `busctl` that select the user bus."
  - "internal/shellguard/rows.go `hubTmuxServerCommandPattern`: probe ordinary tmux spellings of `-L` and `-S` (flags clustered with others, quoted paths, a quoted or spaced directory, other argument words before the flag) for misses and false denials, and settle whether `lym-` matches the entry's \"another hub's server\" as the answer key intends."
  - "internal/shellguard rows, `Forms`, `Notice` and `Steer`: check that each new row agrees with how sibling rows are consumed (both engines, the announced notice, any steer or form checks), that each steer is true for the sessions that read it, and that docs describing the installed denies move in the same diff."
  - "PATTERN.md and pattern/: check the diff against the repository's rules, among them the import limits on `shellguard` given that its integration test now imports `reedengine`, and the package doc's own statement of what it imports."
  - "Tier-2 tests in internal/reedengine (switch, buttonclick, remotecontrol): check that the two-client setup would fail on the old `display-message -c` read, and that no assertion or limit existing at the review base was loosened, here or in the claudeengine settings tests."
  - "_lym/landing/summary.md: check its claims against the diff, including the `## Merge danger` door and whether the blast radius names everything the diff can break."
---

Seed focus for round 1.
The diff has two independent parts, the client-session reads in `reedengine` and two new `shellguard` rows,
so the entries cover each part and the claims made about both.
