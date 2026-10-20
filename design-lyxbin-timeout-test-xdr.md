# lyxbin TestRun_KillsTheGroupOnTimeout fails when Cortex XDR holds the sleeper binary past the 5 s timeout

`lyxbin-timeout-test-xdr` · Task · bug, testkit

Imported from [issue #767](https://github.com/Knatte18/loomyard/issues/767).

Seen 2026-10-20 on OSL-1046 with Cortex XDR, in the first cacheable integration run of the trimpath before-measurement: internal/testkit/lyxbin TestRun_KillsTheGroupOnTimeout failed because its sleeper never wrote its pids within the 5 s timeout. XDR holds every executable with an unseen hash for 8-10 s before it runs, so a fresh helper binary can miss a 5 s deadline. Other runs passed. Expected: the test does not depend on how fast a freshly built helper starts, for example by waiting for the pid file before starting the timeout clock, or by a deadline that does not include process start.