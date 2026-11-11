# Creel sends an unchanged notice once and drops mail about gone runs

An orch that was down or offline came back to a pile of repeated notices, some about pairs already torn down.
Dobby and fabric posted the same notice on every probe or failed push, and nothing removed mail still unread once its run was gone.
Creel now lets a notice replace an unread older one of the same topic, and a pair's teardown drops the unread mail about its run.

## What changes

- **Repeat notices are replaced, not stacked.**
  A post can carry a topic.
  A post with a topic returns undelivered the sender's unread, unreserved mail to the same receiver under the same topic, with the journal reason `superseded`.
  The receiver reads only the newest notice of each topic, and a wait on a superseded message's reply is answered returned.
  Mail already read or being read stays.
- **Mail about a gone run is dropped.**
  A post can carry a run, the slug of the run it concerns.
  The new `drop-run` verb returns undelivered every unread, unreserved message tagged with a run, whatever its sender and holder, with the reason `run-gone`.
  `pairteardown` sends it once a pair is removed, through a run mail dropper that `cmd/lym` installs with `orchcli.InstallRunMailDropper`.
  A failed drop logs a Warn and never fails the teardown.
- **Who tags what.**
  - Dobby's run notices carry the topic `dobby-run-notice/<slug>` and the run `<slug>`.
  - Fabric's failed-push notices carry the topic `push/<side>/<branch>`.
    They carry the pair's slug as the run when the branch belongs to a pair, and none for the prime's code branch and the board branch.
  - Batten's notices to the orch carry the run only, since each is distinct text.
  - Other notices stay untagged.

## Design decisions

- The tags are set only through the Go client (`GoClient.PostTagged`), so the CLI's `lym creel post` cannot set them.
  A post to another hub drops them, and retention carries them with the message.
- `drop-run` may only be sent by a program.
  Creel refuses a program claim from the pane of a strand that is no orch's, so a teardown run from there drops nothing.
- `GoClient.DropRun` answers none without starting a daemon when the hub has no creel journal directory.
- Supersession is scoped to the same sender, receiver and topic, so one sender's notice never retires another's.
- Fabric's failed-push bookkeeping still marks a record notified once and mails again only on a new failure.
  `PushNotifier` now takes a `PushNotice` carrying the line, topic and run.
- The records branch's pair slug comes from the paired code worktree's base name.
  The prime's records branch maps to the prime's worktree name, which no pair's teardown shares, so it is never dropped.

## To check by hand

Nothing here was run against a live daemon outside the tests, and no regression check ran.
A reviewer should look at these points:

- The tags a caller sets on dobby's and fabric's notices are covered only where the integration test pins them, so a caller that sets its tags differently may not be caught.
- The creel wire path for tags and `drop-run` has little direct test coverage; check the client-to-daemon round trip.
- `pattern/PATTERN-pair-remote.md`: the new bullet on dropping run mail sits next to the claim-settle bullet and may split it.
- The fabric push notifier's tags are not asserted in every wiring test.
- The check that the run notices' `Post` seam is filled covers `RunNotices.Post`; confirm no other seam can be left unfilled.
