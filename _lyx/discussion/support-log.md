# Support log — gate-tier-verbs

## Interview

A darn run holds no interview: the writer read the board entry and the gate code and made the calls itself.

## Rejected alternatives

- **`--upto` exclusive with `--tier`.** Rejected: it leaves no one-call way to add a manual tier to a range.
- **Refusing an `--upto` above the highest declared tier.** Rejected: "every non-manual tier" then needs the caller to know the highest number.
- **An "every tier" flag.** Rejected by the operator decision in the board entry.
- **Writing the slot number into the holder record.** Rejected: older records lack it, and the file name already carries it.
- **Refusing `lyx gate status` outside a hub.** Rejected: an unslotted worktree has a true answer, no slots, and a read-only verb need not fail.
- **Listing waiters in status.** Rejected: wait records are per worktree, so a hub-wide view would have to walk every worktree, beyond what the board entry asks.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Does `--upto` combine with `--tier`? Self-pick: yes, as a union.
- Q: May tier0 be manual? Self-pick: no, refused at load.
- Q: May a gate list name a manual tier? Self-pick: yes.
- Q: What does status answer outside a hub? Self-pick: ok with `in_hub: false`.
- Q: Does status show the drain? Self-pick: yes, as a boolean.
