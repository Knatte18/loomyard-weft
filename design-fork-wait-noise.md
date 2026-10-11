# Webster forks wake Merriam on every gate wait

`fork-wait-noise` · Task · enhancement, webster, stencil · running · PR-Gate

## Seen (operator, 2026-10-11, pane-inject-safety's Webster run)

Merriam's pane fills with interim notices for one batch fork, each one a Merriam turn that does nothing:

```
● Agent "Implement batch 01" finished · 4m 39s
The fork is now on card 4's gate, so the batch is progressing and not done. I'm waiting for its final notification.
● Agent "Implement batch 01" finished · 11m 50s
The fork is on card 6's gate, still working through its cards. I'm waiting for its final notification.
● Agent "Implement batch 01" finished · 11m 57s
The fork has reached the batch gate. I'm waiting for its final notification.
● Agent "Implement batch 01" finished · 12m 16s
```

A fork that starts `lyx gate test` as a background Bash call ends its turn while it waits, and Claude Code reports the sub-agent as finished each time it stops with no live background child of its own.
Every such notice wakes Merriam, which re-reads its whole context from cache to answer that it is still waiting.
The orch sees the same pattern from its own agents.

Inside the fork, the background calls carry noise of their own, for example:

```
● Background command "cat /tmp/claude-1000/-home-knatte-Code-loomyard-LYXHUB-pane-inject-safety/f9bc3152-…/tasks/bj10g0jpm.output; cd /home/knatte/Code/loomyard-LYXHUB/pane-inject-safety && lyx loom lint-comments 2>&1 | tail -5; lyx loom lint-pattern 2>&1 | tail -8" completed (exit code 0)
```

- Quick checks (`lint-comments`, `lint-pattern`, reading a finished task's output file) run as background calls, so each adds a notice and a turn.
- Commands carry long absolute paths: the task output file under the system temp directory, and `cd <absolute pair path>` while the fork already works in that pair.

The interim wakes do not keep Merriam's cache warm in any way it needs: the hub's `shuttle.yaml` gives the `webster` role a one-hour prompt-cache lifetime, so Merriam's cache outlives any batch under an hour without them.

## Wanted

- One notice per batch fork: the fork waits for its gate inside its turn instead of ending it, for example with a capped in-turn wait like `lyx shuttle await` on the gate's result, so Merriam wakes only for the fork's final report.
- Forks run quick checks in the foreground; only the gate run that can outlast the Bash timeout goes to the background.
- No `cd` into the pair the fork already works in, and no long absolute paths a relative one would do.
- Fix it where the fork's directive and the Master template say how to run gates, so every fork and Merriam follow it; check whether the orch's own agent briefs should say the same.
