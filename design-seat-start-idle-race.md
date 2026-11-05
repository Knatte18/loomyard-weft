# Seat start fails with 'pane is not idle' while the pane shows an idle prompt

`seat-start-idle-race` · Task · bug, shuttle, burler · running · Code-Burler-Review

Imported from [issue #50](https://github.com/Knatte18/loomyard/issues/50).

What happened: step Plan-Burler-Review (burler-review) halted as failed with anomaly producer-hard-failure. Round 1 ended with two consecutive died outcomes (attempt 1 and attempt 1b), each "review never started".
Both attempts failed in seatengine with `shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: ... the pane is not idle; retry once the session is idle`, then `shuttle: the provider never became ready`.

Why it is abnormal: the saved pane capture in both attempts shows the skill-loading turn finished (the three Skill calls succeeded, the agent replied `ok` with the skill names, the status line read "done") and an empty `❯` prompt on screen. The session was idle, yet the idle check refused. This looks like a race between the end of the skill-load turn and the idle detection used before delivering the prompt. The second attempt failed the same way a few seconds later, so the retry does not clear it.

What to do: make the idle check after skill loading robust against the post-turn settle (re-poll for a bounded time before refusing, or accept the empty prompt plus a completed-turn marker). Consider a retry with backoff inside the seat start before counting the attempt as died, so one transient refusal does not consume a review round.

Evidence: shuttle run dirs under `.lym/shuttle/`, `startup-capture.txt` in each; trace file under `.lym/logs/`.

Provenance: task server-based-live-viewer; notes loom-halt.md.
