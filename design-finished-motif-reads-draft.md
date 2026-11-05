# A finished motif run leaves its entry as an ordinary draft, so it reads as not done

`finished-motif-reads-draft` · Note · bug, board, loom

Imported from [issue #53](https://github.com/Knatte18/loomyard/issues/53).

Seen in hub Models-LYMHUB, beam duplication-and-import-followups, 2026-11-04.

The motif task knut-rheology-friction-pvt ran Discussion to an approved decision record. lym batten status reports state done, the record is stored on the entry (lym board get --record works), and the entry carries discussed: true. But the board status went back to draft, so the board list and the rendered README show it exactly like a task nobody has started. The operator read it as unfinished work, and nothing on the board says the motif is done.

Related: the entry is also not treated as finished by the board's own checks; see #52, where a project move is refused because the board still believes the entry has a pair.

Expected: a motif whose Discussion is approved shows as finished on the board, for example a distinct status such as discussed or done-motif in the list, the --text listing and the README, rather than a plain draft. When the record is meant to feed a later task, the entry still needs a visible finished state and no pair record.

Workaround: none; the operator has to be told the draft status is how a finished motif looks.