---
round: 3
focus:
  - "tools/machinecost/newbinary.go, linkProbeProgram and its callers: on SIGINT to the process group, and on SIGINT, SIGTERM or SIGHUP to machinecost alone during a go build, does any go, compile or link process outlive the probe, is every folder the probe made removed, does the probe stop with the interruption reported, and do the code comments and package doc state what happens?"
  - "tools/machinecost, every go command the probe starts (the build in linkProbeProgram and any version or other query): run with a scratch HOME and no Go cache or config folders, does any of them create or write a file outside the -dir folders and the probe's temp folder, can an inherited variable such as GOCACHE, GOCACHEPROG, GOENV or XDG_CONFIG_HOME redirect those writes, and does the build still succeed and stay outside the timed runs with a cold cache inside the temp folder?"
---
Both entries concern the probe's `go` child processes: their lifetime on interrupt and the places they write.
