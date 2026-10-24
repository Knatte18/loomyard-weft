# Support log — decision-add-batch

## Interview

A darn run holds no interview: the writer read the board entry and the code and made the calls itself.

## Rejected alternatives

- Repeated `--title`/`--decision`/`--rationale` and `--pattern-*` flags paired by position: rejected because positional pairing of repeated flags breaks silently when one flag is left out, and multi-line texts stay hard to quote.
- One commit per entry inside a single call: rejected because the board entry asks for one commit.
- A short notice for every receiver, Merriam included: rejected because Merriam reads no decision record, so a batch would never reach it.
- A short notice for one-entry calls too: rejected to keep this change to the batch form; the one-entry notice is not the noise the board entry names.
- A markdown batch file: rejected because a JSON array needs no new parser and its keys match the flags.

## Review rounds

_No rounds yet._

## Question ledger

- Which batch input form? Self-picked: `--from <file>` with a JSON array (resolved).
- Does `--amends-plan` work with a batch? Self-picked: yes, every title is appended (resolved).
- Does Merriam get the texts? Self-picked: yes, inline in its one notice (resolved).
- Does the one-entry notice change? Self-picked: no (resolved).
