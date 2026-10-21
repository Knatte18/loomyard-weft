# Support log — shellguard-fixes

## Interview

A darn run holds no interview; the board entry and its two source issues, #770 and #772, are the whole input.

## Rejected alternatives

- A Go-side hook that parses the command with a shell parser: exact, but it makes every Bash call spawn `lyx`, adds a parser dependency and replaces the table's grep -E contract.
- Fixing only the raw go row's prefix: leaves two notions of command position in one table, and the redirect row needs the quote-aware scan anyway.
- Checking whether a redirect target is tracked with `git ls-files` in the hook: the hook would have to pull the target out of JSON in sh, which is more fragile than the static rule.
- Allowing an absolute path holding `/.scratch/`: grep -E has no negation, and a pattern for a word not containing a substring is long and easy to get wrong; the relative path is the way the directive names.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Does the quote-aware prefix apply to every row or only the raw go row? Self-pick: every row built on the shared prefix. Resolved.
- Q: Which redirect targets count as scratch? Self-pick: `.scratch/` by relative path, `/tmp/` and `/dev/`. Resolved.
- Q: Is the orch exempt from the redirect row? Self-pick: no, it gets it like the in-place edit row. Resolved.
- Q: Are variable names still refused inside quotes? Self-pick: only when expanded inside double quotes. Resolved.
