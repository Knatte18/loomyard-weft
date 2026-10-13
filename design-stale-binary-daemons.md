# Long-lived lyx processes restart on a new binary

`stale-binary-daemons` · Note · bug, reed, orch, loom

## Idea (operator, 2026-10-13)

Long-lived lyx processes keep running the binary they started with after a deploy replaces `lyx`, and nothing restarts them.
They should pick up the new binary on their own.

The shared re-exec helper, the status panes and the reed watchdog moved to `reed-window-order` on 2026-10-13.

## Seen (2026-10-13)

- After the watchdog kill, other lyx processes still ran a deleted binary (`/proc/<pid>/exe` marked deleted): `lyx orch watch` (since 2026-10-10) and an operator's `lyx reed attach`.

## Scope to discuss

- Which processes restart themselves: `lyx orch watch`, an operator's `lyx reed attach`, and any other long-lived verb.
- How: each process notices a changed binary and re-execs itself with the same arguments at a safe point (using the shared helper from `reed-window-order`); or the deploy (`update-plugins.sh` and batten's Land deploy) signals or restarts every running process.
- Processes that must not restart mid-work (an agent session, a step in flight) are only reported, for instance in `lyx reed status`.
