---
round: 4
focus:
  - "Does package internal/battenshed build and vet cleanly (go build and go vet on ./internal/battenshed/)? Check every caller of noticeDriverBusy, defined in internal/battenshed/notice.go with a cause error parameter: the one in innerrun.go and the one in publishRetryResume in internal/battenshed/publishretry.go, which must pass the Spawn error."
  - "Does the publish-retry test in internal/battenshed/innerrun_publishretry_test.go still assert the driver-busy notice by its fixed condition now that noticeDriverBusy takes a cause, and does the package's test run pass?"
---

Round 3 found the package not compiling after the merge-in; the focus asks whether the repaired call site and its tests hold.
