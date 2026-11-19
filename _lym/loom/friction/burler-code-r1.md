# Fixing the hardener campaign's code review, round 1

F11's "a copies call carries no green record" lives in the call setup of `internal/gatecli/run.go`, which no unit test reaches without a hub.
The proposed assertion on `exe.green` after `runWithSlots` could not hold, because that function never touches `green`.
I extracted `greenForCall` and added a tier-1 scenario for the call-level check, which cost a full read of the scenario harness.

F13's orphan-kill test failed first because the orphan's working directory sat under the worktree's `_lym` link:
`/proc/<pid>/cwd` shows the link's target, so `namesRoot` (`internal/hardenshed/reap.go`) compares the root against a path it never matches.
A real sandbox process started with that working directory and no root on its command line would escape the reap.
The test now names the root on the command line, where the match holds. I did not change `namesRoot`, since the review did not ask for it;
it is worth a look by whoever owns the reaper.
