# Gate go test runs fill the Go cache with test logs, hundreds of GB a day

`go-test-cache-disk` · Task · bug, gate · running · Describe

## Seen (operator, 2026-11-08)

- `~/.cache/go-build` on neve held about 750 GB when a session of the operator's traced it, and 620 GB while it was being cleaned; OSL-1046 runs the same gates and is likely hit the same way.
- The bulk is the test logs `go test` keeps beside each cached test result: with test caching on, Go logs every environment variable a test reads and every file it opens or stats, to judge later whether the cached result still holds.
  lym's tests build whole hubs with git repositories under `/tmp/lym-1000/...`, so every file operation lands in that log, and one log reaches about 700 MB; over a hundred are written a day, 60 to 275 GB a day.
- The cache never pays back: the temp directories get new random names every run, so no cached test result is ever reused.
  The gate already carries a green record over an unchanged tree itself, so Go's test result cache adds nothing for gate runs.

## Idea

- Gate runs of `go test` run with `-count=1`, which turns Go's test result cache off, so no test log is written; the build cache stays.
- It must need no hub config: the gate puts `-count=1` on Go test runs itself, such as through `GOFLAGS` in the run's environment, or the Go tier template carries it and a reconcile adds it to existing hubs' tier commands.
- Confirm with one run that a `-count=1` run writes no test log into the cache.
