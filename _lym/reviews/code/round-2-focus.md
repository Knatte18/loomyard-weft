---
round: 2
focus:
  - "In internal/battencli, check whether a test drives the driver deps' HandBack.Park seam with a record commit that succeeds and with one that fails, asserting the commit was attempted and the park marker still exists, and whether CommitRecords is asserted to commit with the text batten, run records for, and the slug."
  - "In internal/shedrun/paths.go, check that the holds-a-file check behind DriverRecordsRel omits an absent, unreadable, non-directory or file-less path, and that one exported helper serves it and internal/loomcli/wiring.go with no second copy."
  - "In internal/sheddriver/drive.go finishDone, check that a done stop with friction failed commits the run records after the hand-back report is written, so the report under drive-reports is not left untracked, and that the first CommitRecords call keeps its error handling."
---

Focus targets the places the committed diff touched for run-record commits and the emptiness rule of the driver records pathspec.
