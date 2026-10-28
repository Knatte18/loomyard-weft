# The dobby holds a new run start while the user slice nears its task limit

`dobby-task-limit-hold` · Task · enhancement, batten · running · Darn-Write

Follow-up from task `gate-robustness` (issue #885), which made no .NET build leave MSBuild servers behind and made `lym gate test` rerun a tier once on a fork EAGAIN, but left the dobby unchanged.

With five loom runs and a darn run live, the user slice reached `pids.current 2997` of `pids.max 3000` and every fork failed.
Six parallel runs sit close to the default limit even without leaked processes.

What to do: before a dobby pass starts a new run, read the user slice cgroup's `pids.current` and `pids.max` on Linux and hold the start, logged with both numbers, while the headroom is below a configured margin.
A machine with no readable cgroup starts runs as today.

The EAGAIN retries in git wrappers and the transient halt class `host-resources` for loom runs landed with `transient-halt-recovery`; this note is only the hold on a new run start.
