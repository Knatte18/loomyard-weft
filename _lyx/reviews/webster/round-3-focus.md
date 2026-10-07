---
round: 3
focus:
  - "Comment line breaks: in every Go comment line the branch added or changed, settle whether a clause with its own subject and verb shares a line with the preceding clause after a semicolon or after a comma plus and, but or or. PATTERN-comment-line-breaks requires a break at that boundary; breaks before a participial phrase, a parenthesis or a list item are fine."
  - "Test parallelism in internal/reedengine: settle whether the tests the branch added in strand_test.go (the AddStrandUnless named-strand and no-named-strand tests and their subtests) and in waitmark_integration_test.go (the wait-mark segment test on a real tmux) call t.Parallel, or carry a comment naming the process-global state they touch. PATTERN-test-speed requires one or the other."
  - "Coverage of added behavior: settle whether reedDriverDirectory.DriverRow in internal/loomcli/driverlaunch.go (first row IsDriverStrand accepts, directory error passed through) and the ChildRunLockHeld closure in internal/battencli/wire.go (create lock directory, probe, release a free probe, report a held lock) are reached by a test that exercises their behavior, per PATTERN-test-economy."
---

The focus lists the instances round 2 left open; the reviewer still reads the full range.
