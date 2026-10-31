# Support log — adopt-check-visible

## Interview

A darn run holds no interview; the board entry's Seen and Design sections are the whole statement, read with a coarse reading of `internal/adoptcli` and `internal/reedengine`.

## Rejected alternatives

- A plain line printed after the JSON envelope: stdout and stderr share one writer, and callers decode the output as one JSON line.
- A strand or pane that tails `check.json`: the board entry rules out a strand of its own.
- A poller in reed that reads `check.json`: it adds a wake loop and makes reed learn adopt's record; the check already knows each change.
- Rendering the mark in the prime's session button inside the `#{S:}` loop, so every session shows it: the entry names the prime's bar, and it would crowd every beam's bar.
- Minutes for the time spent, as the wait mark shows: tiers run 40 to 104 seconds, so minutes read 0 or 1.

## Review rounds

_No rounds yet._

## Question ledger

- Where does the plain line go? Self-pick: the envelope's `message` field (D1).
- What does "time spent" measure? Self-pick: the tier in flight's elapsed seconds, plus each done tier's wall time (D2).
- Which process writes the mark? Self-pick: the check itself, through a seam on `checkDeps` (D2).
- Where do the option names and setter live? Self-pick: reed, beside the bar format (D3).
- What clears a stale running mark? Self-pick: nothing beyond the next check and the session's end; bound stated in D2.
