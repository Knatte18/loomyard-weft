# lym gate redundant counts a timeout or memory cap as a kill

`redundant-timeout-not-kill` · Note · bug, gate, testkit

## Seen (orch, 2026-11-17)

The trial `map-tooling-trial` (`a666e8b3`, report `docs/research/speed/map-tooling-trial-2026-11-17/report.md`) ran `lym gate redundant` on `internal/boardengine`.
Of 30994 mutant test runs, 383 ended by timeout and 39 by the memory cap, and the verb counted every one as a kill.
A timeout or memory cap is not proof that the test caught the mutant, so these runs inflate the kills of the tests around a candidate,
which can make a candidate look redundant when it is not; the earlier survey found 27 of 30 sampled confirmations false.

## Idea

Count a run ended by timeout or the memory cap as unjudged, not as a kill, and report those mutants apart;
a candidate whose confirmation rests on such a run is listed as unconfirmed.
