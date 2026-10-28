# Loom halts on transient fork/exec EAGAIN and records an opaque exit status 2 for the sibling halt

`loom-halt-on-transient-eagain` · Note · bug, loom, shed, fabric

Imported from [issue #886](https://github.com/Knatte18/loomyard/issues/886).

Two loom halts were recorded within a second of each other (09:41:18Z and 09:41:19Z), both as `failed`, one in `Drift-Verify` and one in `Code-Bouncer`.

**What happened**

- The `Code-Bouncer` halt (anomaly `producer-hard-failure`) carries this lym error: `shedadapters: Code-Bouncer (bouncer): read the live subject: fabricengine: read task patch: diff the working tree against <sha>: fork/exec /usr/bin/git: resource temporarily unavailable`. That is EAGAIN from the OS while lym spawned `git`, a transient condition.
- The `Drift-Verify` halt (anomaly `interrupted-stop`) carries only `exited: exit status 2`. The note names no cause, and its history lists `Drift-Verify` as `skipped` even though the halt is attributed to it.

**Why it is abnormal**

- A transient process-spawn failure halts the whole run as `failed` with no retry, so a momentary resource spike costs the run.
- The two halts share a time window and most likely a root cause, but nothing in the notes links them.
- The `interrupted-stop` note gives no reason, so the operator must open the trace file to learn what exit status 2 means.
- The Drift-Verify history entry reads `skipped` while the halt names that producer as the failed one.

**What to do**

- Retry `fork/exec` EAGAIN in the fabricengine git wrapper with a short bounded backoff before surfacing it as a hard failure.
- Put the underlying cause (the failing command or its stderr tail) into `interrupted-stop` halt notes.
- Check whether lym itself spawns enough concurrent processes or git calls at the end of `Darn-Write` to exhaust the process limit.
- Reconcile the history state of a producer the halt blames with the halt note.

Provenance: task `review-setup-audit`; notes `loom-halt.md`, `loom-halt-2.md`.
