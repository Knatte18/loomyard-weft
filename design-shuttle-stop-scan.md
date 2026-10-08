# Scan enforcing PATTERN-shuttle-stop

`shuttle-stop-scan` · Note · enhancement, shuttle, reed

## Seen

The PR-Gate review of `shuttle-send` (PR #474) found `PATTERN-shuttle-stop` false against `orchcli`, and no Go test or scan names that entry, so nothing caught it.

## Idea

A static scan allows `RemoveStrand` calls on a shuttle run strand only in `shuttleengine` and the exceptions the entry names.
