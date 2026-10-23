---
round: 3
focus:
  - "Decision D10 in decision-record.md: settle whether every path that can release the parent lock while a Finalize fixer session waits through the landing gate is guarded, covering both batten's loopLockStep idle backstop and the loop's own wake hook in internal/loomcli/looplock.go (loopIdleRelease), and whether Finalize re-checks the lock before landing."
  - "Decision D2 in decision-record.md: settle whether the mechanism for merging the upstream tip, never the local tip, read under the push-skip lock, is decided and in Scope, given that internal/fabricengine's MergeIn takes a branch name and picks the local SHA unless it is a strict ancestor of origin's."
  - "Decisions D11 and D14 in decision-record.md: settle whether the claims about who may lift the landing-stop and reroute-cap halts hold under driver_kind llm, and whether D14's reconcile rewriting changes the SeedOnly profiles.yaml contract of internal/configreg."
  - "Decisions D7, D10 step 4 and D13 in decision-record.md: settle whether the PR-Review producer named exists in internal/shedadapters, whether the unreviewed-fix kind cap covers hubs of every language lyx supports, and whether the stated deploy bound holds when a run's review budget is 0."
---

Focus targets the round-2 design findings; the consistency and scope items are local edits and need no steering.
