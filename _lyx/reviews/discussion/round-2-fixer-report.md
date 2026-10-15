# Fixer report — discussion round 2

## Changes

All four findings fixed as proposed in `_lyx/discussion/decision-record.md`.

- F1: the creel interrupt Acceptance criterion's multi-line clause now pins the sequence (n `C-u` with one `BSpace` between each pair, nothing else) and a fake whose `C-u` deletes only the caret's line and whose `BSpace` at a line start joins the line above; a two-line and a three-line prompt are emptied and a single `C-u` leaves text.
  The premise was checked against `tuiReed.SendKey` in `internal/shuttleengine/fakes_test.go`, which sets `r.box = ""` on any `C-u`.
  D3's sequence text is unchanged.
- F2: D5 now decides that every daemon logger line carries its pid; the client quotes its pid's lines plus pid-less lines after its own header with no later start header, and when another header follows it quotes only pid-tagged lines and says the file holds unattributed lines.
  D5's Bound adds that a truncation erasing only this start's header still finds its later lines by pid.
  The premise was checked against `spawnDaemon` in `internal/creelcli/spawn.go`, which starts the binary directly.
  The Acceptance criterion's wording is unchanged.
- F3: D3's restore-during-paste sentence rewritten as proposed: the window opens no earlier than two settles after the key and lasts from the pre-write guard's read to the paste.
- F4: D1's parenthetical list of box reads now includes creel's interrupt reads before the key, after the settle and after the clear.

## Disputed

None.

## Deferred

None.
