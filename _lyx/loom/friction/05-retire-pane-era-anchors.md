# Retiring the below-parent and top anchors

The card says a `SaveState` of a loaded `reed.json` keeps the retired `size` and `contentRows` keys because `internal/state` sees the old element decode equal to the new one.
That cannot hold: `ReedState` has its own `UnmarshalJSON`, so `internal/state` treats the whole record as opaque and writes it whole; no unknown key survives a `reed.json` rewrite today, with or without this card.
`TestLoadState_RetiredDisplayKeysIgnoredAndLayoutUnchanged` therefore asserts the rewritten file reads `anchor: own-window` and nothing about the unknown keys.
