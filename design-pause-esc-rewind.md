# lyx pause leaves idle drivers in the rewind picker

`pause-esc-rewind` · Task · bug, reed, shuttle · running · Publish

First live test of `lyx pause` (hub scope, 2026-10-12 17:07, grace 10 s).
Both run drivers (`ly:cleanup-1012:driver`, `ly:intake-bugs-1012:driver`) were reported `still_busy` and are recorded as interrupted sessions.
After the pause each driver pane shows Claude Code's rewind picker ("❯ (current) … Enter to continue · Esc to cancel"), the dialog a double Esc opens on an idle prompt.
Both drivers were waiting on a background `lyx shed step` job, so the pause's Esc landed on an idle prompt, or one Esc too many reached it, and opened the picker.
Risk: `lyx resume` types one line into each interrupted session; into an open picker that line is lost or selects a rewind point.

## What to check

- Whether pause sends more than one Esc, or sends Esc to a session whose prompt is idle while a background shell runs.
- Whether resume closes a picker (one Esc) before it types its line, or detects the picker from the pane capture.
- `interrupted` in the pause output was empty while `still_busy` listed both drivers; check what each list means.

Also seen: the shed step processes carried on under the pause (`intake-bugs-1012` Merge-In verify kept running, `cleanup-1012` at Describe); check that this is intended (hold only new agent and network work).
