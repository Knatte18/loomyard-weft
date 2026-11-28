# Notice tests run creel's daemon over the real hub delivery and tmuxrec

The notice tests in `loomcli` and `battencli` ran creel's daemon in process on `creelfake.Delivery`, whose strand list the test wrote by hand.
The PR-Review halt row's unknown-strand refusal in `TestPRReviewGateFailureMail` rested on that hand-written list.
Now the daemon runs over creel's real hub delivery:
a strand is known because reed's state holds it, and its liveness comes from reed reading a recorded tmux.

## What changes

- `creelcli.NewHubDelivery(hubPath, outside)` is the exported constructor of the hub delivery.
  It takes the `reedengine.Outside` each worktree's reed reaches.
  The daemon and `HubStrands` pass a zero outside, so production still reaches real tmux.
  It replaces the unexported `newProductionHubDelivery`; `wireShuttle` builds reed with `reedengine.NewOver`.
- `tmuxrec` recordings hold reed's session name as the `{session}` placeholder.
  A session the kit builds binds it to `pr-session`, so every existing replay sees the same calls as before.
  `tmuxrec.PlayOutside(t, name, session)` returns a replay as a `reedengine.Outside` for a reed of any worktree.
  It binds only the session name, skips on an operating system other than the recording's, and fails the test at cleanup on a call the recording could not answer.
- `startNoticeOrch` in both packages writes the orch strand, plus the extra strands a test names, into reed's state through `reedengine.SaveState`.
  It then runs the daemon over `NewHubDelivery`, with reed replaying `session-not-started`.
  Every strand reads not live, so a posted mail waits unread in the log the tests read.
  The fake's "busy" scripting had done that job before.
  The loomcli helper now takes strand names rather than `creelengine.Strand` values, and returns nothing, since no caller read its results.
- The stand-in inventory row for these tests records the replacement as done.

## Design decisions

- **The recorded tmux kit, not real tmux.**
  The tests run at tier1, which allows no tmux server.
  None of them spawns tmux through reed today, so the replay keeps them free of the tmux binary.
- **The placeholder went into the stored recordings without re-recording them.**
  Every recorded occurrence of `pr-session` became `{session}`.
  None of them sits in a call's standard input or in a reading, so the files now hold what the recorder writes with the new binding.
  Their source lines stay as recorded.
- **The `creelfake.Clock` stays**, as the stand-in inventory rules for a clock.
- **Out of scope:** `orchcli`, `beamcli`, `reedcli` and `dobbyengine` still run their own copies of the daemon on `creelfake.Delivery`.
  They can move onto `NewHubDelivery` and `PlayOutside` the same way.

## Merge danger

**Door:** two-way
**Blast radius:** creel

The production change is a constructor rename plus `reedengine.NewOver` given a zero outside, which behaves exactly as `reedengine.New` does.
The notice tests now skip on Windows and macOS, as the recordings are Linux-only, so they cover less there than before.
