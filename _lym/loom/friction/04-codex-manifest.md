# Card 6 allowlist entry for burlerengine's SeatTables was stale

Card 6 told me to add `internal/burlerengine.Engine.SeatTables` to `testOnlyCodeAllowed`, saying no main reaches it before card 8.
The guard reported the entry stale: the analysis already counts `(*Engine).SeatTables` as reached, though only `internal/shuttleparity` calls it.
I removed that one entry in a `6: ... (fix)` commit; `internal/seatengine.Specs` and `internal/landingshed.FixReviewSeatTable` stay as the card says.
Card 8 should remove only those two entries, plus `internal/shuttleengines.NewWithSeams` and `internal/shuttleengine.MissingItems`.
