# lym board upsert --record-file replaces a Sketch's stored record, stamped with the body hash promote checks

`record-file-upsert` · Task · feature, shed, loom, board · running · Code-Burler-Review

Decision record: [Sketch](decision-record-file-upsert.md)

Filed from the Sketch `decision-record-replace`.

Takes: the board's stored record files and Board.StoreDecisionRecord, the run lock in runlock.go, the Add weft probe in remoteleftover.go, and the strand-name caller classification in boardcli
Delivers: lym board upsert --record-file with its refusals and one-write landing, the record_body_sha256 stamp on every stored-record write, and promote's body-changed warning
Exports: a fabricengine function that reports whether a slug's weft branch is live, which Topology.Add also uses; a discussionparser record-only section check; the record_body_sha256 field on boardengine.Task

The part changes these packages:

- internal/boardcli (about 250 changed lines)
- internal/boardengine (about 250 changed lines)
- internal/discussionparser (about 80 changed lines)
- internal/fabricengine (about 120 changed lines)
