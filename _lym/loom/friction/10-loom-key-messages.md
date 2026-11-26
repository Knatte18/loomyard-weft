# Implementing card 10, loom-key-messages

The card asks for `FixFanKey` to be the full name and attributes it to loomengine and loomcli, but it is set in `internal/shedrecipe`'s `BurlerFix` entry (`opts.FixFanKey = fixFanRef.Key`), which is not a card 10 target.
In production the burler fork refusal and the unrecorded-fan text therefore still name `discussion.fixer_fan` until that line takes `fixFanRef.Text`; card 11 targets `internal/shedrecipe#`.

The first batch gate run failed only on a `lym board` process a `TestLoomPreBootstrap` fixture left running in `internal/loomcli`; the rerun passed with no change.
