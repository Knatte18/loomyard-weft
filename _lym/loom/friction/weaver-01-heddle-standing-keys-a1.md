# Implementing cards 1-3 of the launch manifest plan

Card 2's gate could not pass as written.
It creates `shuttleengine.WriteManifest`, whose first production caller arrives in card 3, but lists an interim `testOnlyCodeAllowed` entry for `MissingItems` only, so the reachability guard failed.
I added a second interim entry in card 2 and removed it in card 3, whose targets do not name `cmd/lym#testOnlyCodeAllowed`.
