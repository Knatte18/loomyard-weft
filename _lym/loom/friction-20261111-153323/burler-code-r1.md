# Friction, code review round 1 fix

The review's F2 fix quietly changed what a test helper did: the three `RetiredBeams` rows used a button of an unlisted beam as their wake trigger, and the retire change now drops that button.
One row turned flaky under load and cost two gate runs to trace.
The review named the keeper and relay tests to update but not those rows.

`TestRunRelay/WatchThatFailsToOpenFallsBackToTheOnDiskCheckWithItsOwnBackoff` failed once under `-count=5` with pending waits `[8s 10s]`: the 1s, 2s and 4s advances cross the 5s outbound wait, so an outbound wake can drain the hint first.
It is a race of that existing row; alone it passes 40 of 40.
