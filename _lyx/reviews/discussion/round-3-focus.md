---
round: 3
focus:
  - "D2 in _lyx/discussion/decision-record.md: for a reader segment, check whether every spelling that can run code under bash or zsh is rejected by the command-wide checks D2 names as its bound, in particular quoting forms such as ANSI-C `$'...'` that `blankQuotedSpans` in internal/fabricengine/refscanner.go mis-parses, and zsh parameter forms such as `$~x`, `$=x`, `$^x`, `$+x` and positional parameters; settle whether the dollar rule is stated so that it denies by default rather than listing followers."
  - "Acceptance criteria in _lyx/discussion/decision-record.md: check that each rule D1 and D2 introduce has a row that fails under a plausible wrong implementation, including the dollar rule in a double-quoted span and on special-parameter characters, and D1's counting of an untimestamped main-session transcript with the flag set."
  - "D2 wording against code: check that each claim about what `IsReadOnlyCommand` rejects command-wide matches what the classifier runs, including the stderr redirections it drops before the output-redirection scan."
---

Entries name where to look and which question to settle; they cap no severity.
