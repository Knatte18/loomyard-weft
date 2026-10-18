# creel log: a hold refresh writes a held event every ~2s

`creel-hold-log-spam` · Task · bug, creel · running · Darn-Burler-Fix

Imported from [issue #722](https://github.com/Knatte18/loomyard/issues/722).

Seen 2026-10-18 14:09-14:13Z: while ly:ORCHWATCH held ly:orch during a context cycle, lyx creel log --to ly:orch got a held event roughly every 2.3 s (over a hundred in four minutes), burying the posted/read/injected events. Expected: one held event when a hold starts (and one when it ends or changes), not one per refresh.