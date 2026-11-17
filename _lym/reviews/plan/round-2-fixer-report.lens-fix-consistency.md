```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 fixes to cards 8, 7 and 1 agree with the sibling cards, the overview and the decision record.

no findings

- **Card 8:** no other card or the overview restates the WaitDelay, the "within a second" bound, `sleep 5` or `TestRunPluginHook`. The overview's "Plugin hook processes" decision covers only logging, `runproc` and the killed-`Prepare` bound, so `hookWaitDelay` and the new timeout rows leave no stale copy elsewhere.
- **Cards 7 and 1:** the directory-at-path input now agrees with card 4's `RecordUnwritable` row, which already used that construction. Card 10 still uses a mode-0 directory that skips itself under root. That row needs an unreadable directory for its walk, which a directory standing at a path cannot give. The ruling covers only cards 7 and 1, so card 10 is no twin left behind.
- **Decision record:** the 2026-11-17 entry "Unreadable-file test rows use a directory at the file's path" matches the card 7 and card 1 text as changed.

Outside this lens: card 8's marker row passes `MARKER` through `Env`, and `RunPluginHook` runs with exactly that `Env`. If the test's `Env` omits `PATH`, `sleep` and `touch` resolve only through the shell's default `PATH`. The fix-claims lens owns that question.

Tool calls:
1. Searched cards 00, 02–06, 09 and 10 for WaitDelay, the time bound, the group kill and unreadable or root rows. My context held only cards 1, 7 and 8.
2. Searched the overview for its hook sentences and card 4 for its unreadable row, to read the matches from call 1 in full.
3. Read card 4's Intent and Tests from line 45, to see what input its unreadable and unwritable rows name.
