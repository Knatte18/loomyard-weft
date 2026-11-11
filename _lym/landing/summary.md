# Creel keeps the newest unread notice per topic, drops gone runs' mail

Two creel faults showed up while the orch was away from its mail.
During a network outage, the orch's mail filled up with about 25 dobby run notices for the same runs, and FABRIC sent the same push failure for one branch several times.
Then, after `lym cleanup --apply` removed some pairs, the orch was handed day-old mail about those runs as if it were live, including a request to settle a stop of a run that was already gone.

Both senders already send each notice only once per episode: dobby keeps episode keys in a marker, and fabric keeps a `Notified` flag on the failed-push record.
That was not enough, because a new episode starts whenever an agent's activity moves or a push record clears and fails again.
The real cause is that creel kept every unread notice for an orch whose strand was down, and delivered them all once it came back.
Mail from a program to an orch is never returned by the sweep, so nothing removed notices about runs that were gone.

## What changes

**Notice tags in creel** (`internal/creelengine`, `internal/creelcli`).
A post can carry `PostTags{Topic, Run}`, which only the Go client sets.
- **Topic.** A post with a topic returns undelivered the same sender's unread, unreserved mail to the same receiver under the same topic, with the reason `superseded`. A reply wait on such a message is answered returned. The receiver therefore reads only the newest notice of each topic. Mail that is already read, or being read, stays.
- **Run.** `drop-run` is a new Go-client verb that only a program may send. `Store.DropRun` returns undelivered every unread, unreserved message tagged with the named run, whoever sent it and whoever holds it, with the reason `run-gone`. An empty run is refused.
- **Retention** carries both tags with the message.
- **Client.** `GoClient.PostTagged` posts with tags. `GoClient.DropRun` sends nothing when the hub has no creel journal yet, so it never starts a daemon in a hub that has no mail.

**Senders tag their notices.**
- **Dobby run notices** go through a new `RunNoticeDeps.Post` seam. The hub wiring tags each one with topic `dobby-run-notice/<slug>` and run `<slug>`.
- **Fabric push failures.** `PushNotifier` now takes a `PushNotice{Line, Topic, Run}`. The topic is `push/<side>/<branch>`. The run is the pair's slug, read from the base name of its code worktree; the prime's code branch and the board branch carry no run. The remote-teardown notice carries topic `remote-teardown/<slug>` and no run, since it asks for a step that is still owed after the pair is gone.
- **Loom, burler and webster notices from a pair** go through `orchcli.NotifyParent`, which tags them with the pair's slug as their run.
- **Batten's run notices** are tagged with the run's slug. When the pair is absent they go through the new `orchcli.NotifyRunFrom`.

**Teardown drops the run's mail** (`internal/pairteardown`).
After a successful removal, `RemovePair` calls the process's run mail dropper with the slug. A failed drop logs a Warn and never fails the teardown.
`orchcli.InstallRunMailDropper` installs the dropper from the root pre-run, the same way the push notifier is installed. It signs the drop as the `TEARDOWN` program.
`pattern/PATTERN-pair-remote.md` records the new step.

## Design decisions

- **Collapse in creel, not in each sender.** The duplicates pile up only while the orch has not read them, and only creel knows that. A topic keeps the newest state rather than the first one, and it handles every sender the same way.
- **Supersede is limited to one sender.** A post replaces only its own sender's mail under a topic, so no sender can remove another's mail. A FABRIC notice posted from inside a non-orch strand's pane is signed as that strand, so it collapses only with that strand's earlier notices.
- **Teardown drops through an installed hook.** pairteardown cannot import `creelcli`, because creelcli's tests import pairteardown through reedcli. It therefore exposes `InstallRunMailDropper`, and the root pre-run installs it.

## Known limits

- A teardown run from the pane of a strand that is not an orch drops nothing, because creel refuses a program claim from such a pane. That refusal is logged.
- With no dropper installed, which is the case in tests, nothing is dropped.
- Mail posted untagged by other routes, such as `NotifyFrom`, `NotifyAsker` from a beam, or posts made by agents, is never dropped by a teardown.
