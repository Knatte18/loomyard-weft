## Interview

A darn run holds no interview; the board entry is the whole brief.

## Rejected alternatives

- Writing outputs next to the script: rejected, the entry forbids anything generated under `docs/`.
- Committing `summary.md`: rejected, the entry names `runs.tsv` and `forks.tsv` as the baseline and the summary result goes into the README.
- Finding the repo root with `git rev-parse --show-toplevel`: rejected, a fixed relative climb needs no git and matches the original script's idiom.

## Review rounds

_No rounds yet._

## Question ledger

- Which default for the records repo? Self-pick: keep `$REPO/../loomyard-weft`, document the `WEFT` override. Resolved.
- Whether to run the full script before committing? Self-pick: no, it re-parses 50+ archive tags and transcripts; a syntax check and path read suffice. Resolved.
