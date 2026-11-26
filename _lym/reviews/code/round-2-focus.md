---
round: 2
focus:
  - "Cleanup on interrupt and termination: follow runProbeCommand in tools/machinecost/main.go through runProbe and each measure function. Does every path that makes a folder inside a -dir folder or the temp folder still delete it when a signal arrives, including while a child process or go build is running, and does the package doc claim only what the code does?"
  - "Child environments: read the environment built for go build in tools/machinecost/newbinary.go, for the git workload in tools/machinecost/gitload.go and for the timed process starts. Does each doc comment's claim about what is pinned or ignored hold, does any variable the child needs get dropped, and does anything get written outside the -dir folders and the probe's temp folder?"
  - "The -dir argument contract: check dirList.Set in tools/machinecost/main.go against the package doc, the flag usage text and the measurement name shape, for empty names, names holding /, paths holding = and relative paths."
  - "Tests: in tools/machinecost/cli_test.go, probe_test.go, newbinary_test.go and report_test.go, would each test of the -dir flag, of cleanup after a failed probe, of a fresh build ID per linked program and of the throttle count sum fail if the behavior it names were removed?"
  - "Named constants: across tools/machinecost, tests included, is any name or path that can go stale still written as a literal in more than one place, as PATTERN-code-conventions forbids?"
---

The entries point at the areas where this round found problems, so the next round reads them as the diff now leaves them.
