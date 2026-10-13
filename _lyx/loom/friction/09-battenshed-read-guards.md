# Implementing cards 9 to 13 (battenshed, shuttle, orch and shedverbs log guards)

Three cards described test behavior the code does not have, and each needed a small adaptation.
Card 10 listed "draft that never clears fails busy" as a quoted-box row, but its engine reads no input box, so the row got `reader: true`.
Card 11 gave the draft-row `wantActivityWarns` as 1, but a draft hold returns before the activity query, so that row stays 0.
Card 12 said the unreadable network-operation marker list is asked again on every grace tick, but the first failed read marks the pause entry handled, so the recovery Info appears only when the next entry asks, and the row resumes the first entry and pauses a second.
