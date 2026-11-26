# Card 11: renaming keys in Go messages across modules

The card's Intent claims every further message holding a bare key, but its Edit list names a fixed set of packages and the card gate runs only those.
Asserting tests outside the list therefore fail unseen by the gate (burlercli, shuttleengines, configcli, loomengine), and I found them only by testing extra packages.

The card is not complete outside its target packages: bare key names remain in messages and help texts of
`gatecli`, `batcher`, `battencli`, `beamcli`, `webstercli`, `hibernatecli`, `reportcli`, `preflightshed`,
`weaverengine` (`ResolveLadder` and `settingRefusal`), `sheddriver/tackler.go`, `shedarbiter`,
and in `burlerengine/fix.go` and `loomshed/plancontext.go` (card 10's packages).

The settings golden in `claudeengine/testdata` is too large to edit by hand and has no update flag;
I rewrote it with a temporary test, run once through the gate and removed.
