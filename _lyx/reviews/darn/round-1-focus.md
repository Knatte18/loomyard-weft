---
round: 1
focus:
  - "tools/tokencount/shed.go and count.go: does fullLoomRun match the board entry's definition of a full loom run (newest archive tag, seed recipe, Finalize in history), and does recentRuns return counted and skipped slugs correctly for -last N, named slugs and the no-run error?"
  - "The diff touches internal/battenshed and internal/loomcli, which the summary never mentions: does the board entry license those changes, and does the summary account for them?"
  - "Docs in the same diff: package doc, tokencount.sh/.cmd comments and docs/overview.md against the repository's constraints document; any claim in the summary the diff does not bear out."
  - "Test economy: does TestRecentRuns or any new loomcli/battenshed test duplicate existing coverage, and is each added behavior (skipped-slug line, header wording, error text, inFlight) covered?"
---

Seed focus only; the reviewer still reads the whole committed diff.
