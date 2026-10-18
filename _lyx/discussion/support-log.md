# Support log — creel-compact-notices

## Interview

A darn run holds no interview: the board entry and its operator decisions of 2026-10-18 are the whole input.

## Rejected alternatives

- Packing every pending notice into one typed line: a line would grow without bound and one busy send would hold them all back; one notice per probe reuses the loop as it is.
- Journaling the notice read before typing it: a busy send would then lose the notice; journaling after the send risks only one retype after a crash.
- Inlining a multi-line program message by flattening it: the flattened text could misread, and the read verb shows it intact.
- Treating EXTERNAL as a notice sender: an EXTERNAL sender waits for a reply with `post --wait-reply`, so its mail wants the id and reply hint.
- Keeping `--all` as a no-op: an unused flag is a compatibility shim for internal callers, which change with it.
- Dropping the sender from a program message's read header: the body does not always say which program wrote it.

## Review rounds

_No rounds yet._

## Question ledger

- Which senders' mail is a notice? Self-pick: a program mailbox name, never EXTERNAL or a strand.
- What happens to a long or multi-line program message? Self-pick: it stays mail behind the contentless reminder.
- How old is "more than a few minutes"? Self-pick: five minutes, by the CLI's clock.
- Does `read` keep a JSON form? Resolved by the board: behind `--json`.
- Does the default read return all senders? Resolved by the board: yes, `--all` goes.
