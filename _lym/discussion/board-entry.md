# Board entry `compact-inject-draft`: Watcher's /compact inject into an idle orch is typed but not submitted (unconfirmed, then stalled on draft)

Imported from [issue #897](https://github.com/Knatte18/loomyard/issues/897).

## Seen

On 2026-10-28 the orch watcher's `/compact` cycle inject into the idle orch strand `ly:orch` was typed but never submitted.
The orch had written its note and ended its turn; the session was idle.
`lym creel log --since 2026-10-28T13:35:00Z` shows:

- 13:44:05.80 `posted` inject #107, command `compact`, from `ly:ORCHWATCH`.
- 13:44:05.93 `sending` #107.
- 13:44:25.52 `unconfirmed` #107: the send answered not landed, so creel never retypes it.
- 13:44:43.53 `posted` inject #108, command `color`.
- 13:44:43.65 `stalled`, reason `draft`: the `/compact` text sat in the input box unsent.

The cycle stalled until the operator submitted `/compact` by hand.

## Suspected cause

Unconfirmed: `creel-hook-post` (161a4a7f) had been deployed about twenty minutes earlier.
It added an `asyncRewake` `Stop` entry, `lym creel hook-wait claude`, that stays open while the session is idle.
The typed line or its Enter may race that hook, or the pane may not read as ready for the verified send while it runs.

## Workaround

The operator submits the stuck `/compact` draft by hand.

## Expected

A cycle inject into an idle orch is submitted, or an unconfirmed send whose text sits as a draft is completed or cleared and retried, so the cycle never hangs on a draft.
