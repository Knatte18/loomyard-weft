# Support log — plan-writer-audit-scripts

## Interview

No interview: a darn run holds none.

## Rejected alternatives

- Writing outputs next to the scripts, as the source did: a rerun would overwrite the committed baseline.
- Expecting a prebuilt `tokencount-bin` in the folder: a binary may not go into git, and a build on first use is what `compare.sh` already does.
- Rewriting the scripts to drop the `extra-loom.tsv` dependency: it would change what a rerun measures; the file is optional instead.
- Regenerating the baseline tables by a fresh run: the task keeps the 2026-10-24 measurement.

## Review rounds

_No rounds yet._

## Question ledger

- Where does `extra-loom.tsv` come from when it is not committed? Self-pick: optional input under `$OUT`; the README says how a run without it differs.
- Which default for `OUT`? Self-pick: `$REPO/.scratch/plan-writer-audit`, as the task states.
- Should `results.md` be updated to the new paths? Self-pick: no, it is the baseline; the README holds the current commands.
