---
round: 2
focus:
  - "internal/loomcli/prreview_mail_integration_test.go, prReviewMailDeps and its callers: for every seam, record, config value and output the test supplies, does it come from the owning module or is it hand-written, as PATTERN-test-stand-ins judges it, and do the expected head and mail text have a source independent of a literal stand-in?"
  - "internal/loomcli/prreview_mail_integration_test.go, TestPRReviewGateFailureMail, against the three mail rows that TestPRReview_Wait in internal/loomcli/prreview_wait_test.go held at the review base b439f157e432a2f6559d9fbf5e68daa2aae9995f: does every assertion the shared loop made for those rows, the push count and the whole record included, still hold somewhere no looser?"
  - "internal/testkit/tmuxrec: in record.go (placeholdCall and the binding of a call's Stdin) and session.go (newSession's handling of a shell that does not resolve), does replay match what the code sends on a host whose bound values differ from the recording host's, and does a test pin each behaviour the diff adds there?"
  - "internal/testkit/envkit/envkit.go: do the session and strand seams FullEnv and the batten child fill agree with the scope the board entry gives for moving envkit's session stand-ins, and does _lym/landing/summary.md describe what was and was not moved?"
  - "docs/research/no-fake-other-modules/loom-landing.md and _lym/landing/summary.md: do the inventory rows and the Merge danger blast radius match what the diff over b439f157e432a2f6559d9fbf5e68daa2aae9995f..HEAD removes, changes and reaches, internal/reedengine/io.go included?"
---

Round 2 reads whatever changed since round 1 as new text, so each entry asks its question over the current diff.
