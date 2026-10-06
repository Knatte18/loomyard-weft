# Round 2 fixer report

All four findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` unchanged.
Round 1's fixes did not regress; its F3 claimed D4 covered the hook case, and this round made that true.

## Changed

- F1 (D1, D4): D4 now states that the assertion read also asks whether the candidate makes git run module code through a hook, alias or helper, and keeps such a candidate under D2.
  D1's wording now points at that question as stated.
- F2 (D3): a run with a failed package still writes the other measured sections, and a measured package with no existing section gets a new one in import-path order.
- F3 (D2): a whitespace-only reason counts as empty, and the directive's next line must start a top-level `func Test…`.
- F4 (acceptance): the build-failure bullet was split from the removal and refusal bullet and now says "list or build" with the non-zero exit.
  New scenarios cover a section added in import-path order, a directive on a non-candidate listed under "Kept" with a kept test staying among the covering tests, and a whitespace-only reason.

## Deferred

None.
