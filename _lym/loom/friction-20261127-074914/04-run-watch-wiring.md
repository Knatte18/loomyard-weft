# Implementing batch 04-run-watch-wiring

Card 1's addition of `beam.run_watch` to the `lym board upsert` help pushed it past the 2000-byte cap that `TestCLIForm_EveryCommand` in `cmd/lym` holds.
Neither batch 01's card gates nor its batch gate ran that test, so it first failed at this batch's `--main-packages` gate.
It was fixed in `f2b886f11`.

Foreground `lym gate test` calls with a 3600000 ms timeout were moved to the background by the harness after 180 s, several times.
Each time the call then ended with exit 75 and had to be run again.
