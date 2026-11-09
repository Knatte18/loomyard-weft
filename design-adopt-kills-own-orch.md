# lym adopt run from the orch is killed when the orch watcher sees the foreign owner

`adopt-kills-own-orch` · Note · bug, orch, fabric

## Seen

On 2026-11-09 the orch on Neve ran `lym adopt --all` from inside its own session, to take the hub back from osl-1046.
Step 1's origin reconcile pulled the owner record naming osl-1046 as the prime's owner.
The orch watcher on Neve saw the prime owned elsewhere and exited, with `watcher_exit: owned by machine osl-1046`.
It took the orch session and its reed window down with it.
The adopt command was a child of that session, so it died with SIGKILL during step 5's deploy, in the lym build,
before step 6 wrote this machine as the owner.
The prime stayed owned by osl-1046, with no orch and no dobby running on Neve.
The operator recovered by running `lym adopt root` from a plain shell.

## Idea

An adopt in progress must not trip the orch watcher's owner check:
the watcher could skip its exit while an adopt of its subject runs on this machine,
or adopt could write the owner before anything that lets the watcher read a foreign owner.
Until then, the orch rules should say to run `lym adopt` from a shell outside the orch session.
