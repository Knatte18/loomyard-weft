---
round: 2
focus:
  - "internal/gitrepo/blobread.go WriteTreeAtRevision and writeBlobFile: does a blob stored as 100755 reach the base checkout with an executable mode bit, and is a symlink or submodule entry still skipped? Settle it against go-git's filemode predicates, and check the doc comments of WriteTreeAtRevision and fabric's ExportCodeTreeAt against the behavior."
  - "internal/gatecli/redfirst.go and internal/gateslot/config.go: does any code path in the diff branch on the scoper literal \"dotnet\" or \"go\" outside a table keyed by scoper name, as PATTERN-code-conventions \"No hard-coded choice\" forbids? Check that each scoper's red-first behavior is one table row and that the Go path's messages, envelope keys and exit codes are unchanged."
  - "internal/gatecli/redfirstdotnet.go judgeDotnetRedFirst and dotnetRedFirstCall.run: does a run that exits 0 with no test discovered, and a run that fails at restore or build before any test ran, end in a verdict the writer can act on rather than red-first green or a terminal exit 1? Check the exit codes against redFirstVerdictExit and the exit contract in internal/gatecli/doc.go."
  - "internal/gatecli/redfirstdotnet.go nearestProject and byProject: does an unreadable directory give a named reason instead of climbing to an ancestor project, and do tests of two different test projects each run once in their own project's checkout with only their own filter terms?"
  - "Tests added or changed in internal/gatecli/redfirstdotnet_test.go, internal/gatecli/run.go's checkoutRoot swap, internal/dotnetimpact/source_test.go and internal/gateslot/failedtests_test.go: does each new or fixed test assert what its name states, and does it pin behavior no other test covers, per the repository's test-economy rules?"
---

Round 2 reviews the whole range again; the focus entries name where the round-1 defects sat, for a fresh read.
