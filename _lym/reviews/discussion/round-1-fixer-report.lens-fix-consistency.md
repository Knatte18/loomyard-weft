```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling decisions, criteria and cross-references around them.

**F-a (LOW): the support log still says the archive tags are the list of landed runs.**
- Changed text: D5's rationale, which F6 corrected so it no longer calls the archive tags the list of landed work. D6's new rationale adds: "a tag alone does not mean the run landed".
- Stale twin: the support log's `## Rejected alternatives` bullet still ends "while the archive tags are already the list of landed runs". Question ledger Q1 says "tags already list the runs".
- Summary: the fix corrected the claim in the decision record but left its twin in the support log, which is in the fixer's write surface.

**F-b (LOW): D1's list of kept files credits the landing record and index row to D2 to D4.**
- Changed text: D1's first sentence now reads "holds every copy D2 to D4 write: a PR findings subdirectory, an asks subdirectory, a landing fixer subdirectory, the landing record and the index row".
- Summary: D5 writes the landing record and the index row, not D2 to D4, so the attribution is wrong. The fix added the landing record to a list whose attribution was already loose.

**F-c (LOW): D6's derived-line criterion no longer pins which kind of tag it means.**
- Changed text: D6 now gives a derived line only to a tag "whose history holds Finalize". A tag without an index row that `reportengine` sorts into a partial run is "listed as not landed".
- Stale twin: the criterion "`lym report landed` over a hub with one archived run that has an index row and one without lists both, the second marked derived" was left as it was.
- Summary: a run without an index row is marked derived only when its history holds Finalize. The criterion doesn't say so, so a test built on a cancelled tag would contradict the new "a cancelled run's tag is listed as not landed" criterion.

**F-d (LOW): the PATTERN entry doesn't name the new kept records.**
- Changed text: the `PATTERN-run-records` declared text in `## PATTERN changes` still reads "findings, ask answers, fixer reports and the index row are tracked".
- Summary: D4 now also keeps fix review reports, and D5 adds a tracked landing record, but the declared entry names neither.

**F-e (LOW): D4's skip rule for identical reports leaves out the kind.**
- Changed text: "A report identical to the latest copy of the same row and row entry is not copied again". The naming rule two sentences later sets sequences per "row, entry and kind".
- Summary: the skip rule compares within row and entry, but sequences are per row, entry and kind. It should compare within the same row, entry and kind, so a fixer report is never compared against a fix review copy.

No further tool calls were made.
