# creelengine TestScheduler_Reminders hold case fails on a second Linux host

`creel-reminders-hold-second-host` · Note · bug, creel

Imported from [issue #744](https://github.com/Knatte18/loomyard/issues/744).

Seen on a Linux host (git 2.43.0, tmux 3.4, go 1.26.9) with main at c03af73d plus the #739 lyxtemp fix, which creelengine does not read beyond its temp directory.
`lyx gate test ./internal/creelengine` fails twice in a row:

```
--- FAIL: TestScheduler_Reminders/HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry (0.03s)
    deliver_test.go:505: typed ["[creel] #3 from ly:BATTEN: from the owner"]; want only the owner's notice and mail
```

The owner's 4097-byte post under its own hold gets no reminder line; only the notice for the short post is typed.
Not seen before on this host only because no package could run here until #739.

Expected: the test passes on every host, or the reminder for the hold owner's over-long mail is typed as the test wants.