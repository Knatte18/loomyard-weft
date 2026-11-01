---
round: 2
focus:
  - "In tools/tokencount/landed.go, check every error path of findLanding (summaryparser.ParseData failure on the description archived at a run's tag, ChangedLines failure for a root commit): does a single run's failure still abort the whole report, or is the run listed under `No landed commit` with a reason?"
  - "In tools/tokencount/main.go and tools/tokencount/count.go, check how run and recentRuns treat -last at or below 0 and -since: is the behavior of a plain -last 0 or negative -last unchanged from before the diff, or does it now count every finished run?"
  - "In tools/tokencount tests (landed_test.go, main_test.go, time_integration_test.go), check that each behavior the diff adds has a test: the -since run and header, parseSince for a valid date, the zero-changed-lines and unpriced branches of writeRecipes and Cost.precise."
  - "In the header comments of tools/tokencount/tokencount.sh and tools/tokencount/tokencount.cmd, check that the sentence on what the report opens with names every section the diff adds, and that the two wrappers stay worded the same."
---

Focus entries name where to look; the base review of the full range still applies.
