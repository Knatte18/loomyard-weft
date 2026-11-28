---
round: 1
focus:
  - "internal/landingshed/prreview_wait_test.go against internal/loomcli/prreview_mail_integration_test.go: set each assertion of the three removed mail rows (their names, shared table checks such as pushes and entry pushes, record, reason, gate count) beside the new test. Is each still held on the same behaviour and no looser, given the move from tier0 to tier1? For the rows whose no-mail check closures were removed, does the failing MailAgent in prreview_test.go catch the same failure in every one of them?"
  - "internal/loomcli/prreview_mail_integration_test.go: does the mail travel the path production wires, through creel's real client, daemon and refusal, with each stand-in in the test one the repository's rules allow? Do the halt row and the mailed rows each fail if the behaviour their name states breaks, including state shared between rows on one hub?"
  - "internal/testkit/tmuxrec (tmuxrec.go, record.go): follow a paste from recording to replay. Do the buffer-name placeholder, the stdin placeholding and binding (process ids, bound paths) and the match agree on both sides? Check calls with one, two or no buffer names, and the order of replay answers. Does the session-paste recording hold what the code produces, with no host-specific value left in it, and do the new tests and the scenario replay check the stated behaviour?"
  - "internal/testkit/tmuxrec/session.go and internal/testkit/envkit/envkit.go: IdleSession drops the Replay and never skips, and newSession no longer fails on a missing shell. Which callers of LandingDeps and FullEnv, across every platform and tier, can now reach a session seam and get an unanswered or failed call that nothing reports? Is the shell fallback, which also changes PlaySession, licensed by the amended board entry?"
  - "The board entry's third change, envkit's session stand-ins onto the recorded kit: is any session stand-in in envkit, or built for it, left on a hand-written fake? Check the claims that no envkit session sends text and that the remaining closures are listed in the repository's stand-ins inventory."
  - "The repository's constraints document and the docs it names for testkit and test changes: do tier placement, test file naming, import rules, the stand-ins inventory and the package docs match what the diff now does?"
  - "_lym/landing/summary.md: check each claim against the diff, including how the recording was made, and the Merge danger section's door and blast radius against everything the diff touches, internal/reedengine/io.go included."
---

Seed focus from the amended board entry, the commit messages, the change description and the diff outside `testdata`.
The decision record holds only the board entry, with no `Added after Discussion` entries.
