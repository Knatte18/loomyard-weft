# Support log — shuttle-hook-signals

## Interview

A darn run holds no interview: the writer read the board entry, the code and the hub's logs, and made every call itself.

## Rejected alternatives

- Recording `PostToolUse` as a stamp-and-payload line in the events file: tool payloads carry tool output, which would grow the file every reader parses whole on each poll, and a stamp line with no payload would be held back by the session-signal parser as an unpaired stamp.
- Keeping the waiter pid as the liveness test and only adding the stamp: a run whose waiter died would still be skipped, which is the reported bug.
- Telling the parent only, without carrying instructions over: the entry asks for the carry-over and names the notice as the minimum.
- Matching a predecessor by strand name: reed numbers a role against held strands, so the name of a re-spawned row can differ from its predecessor's, and the name is formed only after the prompt is written.
- Carrying over from a done predecessor: a finished session answered its instructions, and every later spawn of a reused role would inherit them.
- Adding `SubagentStart` and `SubagentStop`: see D7.
- Setting `autoContinueAtUsageLimit` in the run settings: see D8.

## Review rounds

_No rounds yet._

## Question ledger

- Is the entry outdated? Resolved: `compact-inject-draft` and `usage-limit-wake` have landed and neither covers these signals; the peers list is empty.
- Why is the waiter pid dead? Open: no waiterless warning appears in the hub's logs, so the cause stays unconfirmed; self-pick: drop the pid as the activity source (D3).
- Which prompts count as the operator's? Self-pick: typed prompts past the prompt offset, minus the prompt itself and every text shuttle sent (D4, D5).
- Which runs are a re-spawn's predecessor? Self-pick: the newest unfinished run of the same role key (D6).
- Should shuttle set the auto-continue setting? Self-pick: no (D8).
