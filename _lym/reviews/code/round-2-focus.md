---
round: 2
focus:
  - "In internal/gatecli/redfirstdotnet_integration_test.go, TestGateRedFirst_Dotnet: commit f8de6d0f7 dropped the row whose real dotnet test tier exited 0 in red-first's exported base checkout (the redFirstGreen path of judgeDotnetRedFirst in internal/gatecli/redfirstdotnet.go). Is that behaviour held on the same path now, by this test or another? If the drop stands, does a Loosened-Check: trailer in the review range name the dropped assertions and a measured cause that compares the parallel run with and without that row?"
  - "In the same test: does each row assert what its name states, including that every test it relies on was selected and run (the envelope's tests key, not only red_tests), and does each row check the exit code that belongs to its outcome?"
  - "Do the test's doc comment, _lym/landing/summary.md (its Measurements section included) and the commit messages of the review range state only figures and cost breakdowns that were measured on the test as it now stands, with its current row count and parallelism?"
---

The entries cover the questions round 1 left open on the one changed test file.
