# Creel hook delivery: mail arrives once, and the HookWake flake

`creel-hook-delivery-fixes` · Task · bug, creel, claude · running · Drift-Verify

## Seen

Two findings on creel's hook delivery, one task.

### Each creel mail reaches the agent twice (operator, 2026-10-29)

The operator sees a creel mail processed twice by the model.
The terminal shows `Ran 2 stop hooks`, then the mail once as `Stop says:` and again as `Stop hook error:`, both with the same `[creel] #<id> from ...` text.
In the orch session each mail reaches the model as two messages: a `Stop hook feedback:` line carrying the mail, and a `Stop hook blocking error from command` message that repeats the mail after the whole hook command text.
The `PostToolUse` variant delivers the mail once, under a tool result.

The shuttle run's `settings.json` registers two Stop hooks:

- a command hook that runs `lym creel hook-read claude` and delivers the mail;
- a second hook that runs `lym creel hook-wait claude` with `asyncRewake: true`, the ring that wakes an idle strand with the bell line `[creel] Mail is waiting for you`.

Which of the two produces the second copy is not yet known.

### `TestScheduler_HookWake` blocked-chain subtest flakes

Run 2 of the `bench-osl1046-xdr` bench, at `tier_parallel` 8 on OSL-1046, failed one `internal/creelengine` test: `TestScheduler_HookWake/MailArrivingInAWokenTurnsBlockedChain...`.
It was not rerun and its message was not read.
It is a third timing-sensitive creel test seen on this machine, after the refused-forward chase subtest and the flakes in `creel-flakes-and-adopt-names`.

## Fix

- Find whether the Stop hook's exit code or output makes Claude Code surface the same text as both feedback and error, or whether both hooks deliver it, and make the mail reach the agent once.
  The hook command text should not appear in the agent's context at all.
- Run the `HookWake` subtest alone repeatedly, with and without CPU load, to find which wait it assumes, and make it wait for the worker or event it depends on instead of for time.
- Check whether the two findings share a cause, and whether the note `creel-ring-carries-mail` is touched by the change.
