---
round: 1
focus:
  - "internal/landingshed/finalize.go, landUnderLock through recordLanding and pushParent: is the single head read the run's own landing commit on every path that reaches it (local merge, a pull request merged on GitHub, a re-run after a partial landing), and do the head-read failure, the nil-seam cases and PushSkipped behave as they did at the base?"
  - "internal/loomcli/landingdeps.go, recordHeldLanding and writeLedgerLanding: the parent branch read now follows the status-records commit under the pair push lock. What state does a failure of that read, or of the ledger write, leave behind, and can a later pass or a push of the pair recover or expose it compared with the base?"
  - "Does writeLedgerLanding, with the landingshed LandLocally seam, deliver the amended entry's Exports line (a writer of one push-ledger landing taking the parent branch, the slug, the landing commit and the records commit, usable for another run), given that it takes a runpusher.Target and reads the branch itself?"
  - "Rewriting a standing landing: through landLocally and runpusher.RecordLanding, are the tip and the earlier records commit kept exactly under the conditions the amended entry states (origin lacks it and the new records commit descends from it)?"
  - "internal/loomcli/landlocally_integration_test.go: do the new test's assertions, including the 'unrecorded last commit' case and the re-record of run one, check the acceptance criterion and what the test's name and doc comment state, or would they also pass if the behaviour were wrong?"
  - "Is the changed behaviour in internal/landingshed (one commit handed to both RecordLanding and LandLocally) covered by a test, existing or new?"
  - "Do doc comments, specs or repository documents outside the diff still describe the ledger landing's tip or the LandLocally argument as the parent's head, where the repository's rules require them to change in the same diff?"
  - "_lym/landing/summary.md: does each claim hold against the diff, including the same-commit claim for a lone run and the Merge danger door and blast radius?"
---
