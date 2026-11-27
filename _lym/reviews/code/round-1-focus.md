---
round: 1
focus:
  - "Session lifecycle in `internal/landingshed/prreview.go` and `prsession.go`: trace every exit (verdict, Stuck, error, context cancel, pause and unpause, interrupt re-entry, session death) and settle whether the session is stopped, resumed or left running as intended, with no orphaned or duplicate session and a halt that `lym loom resume` can recover from."
  - "Stop-before-read and resume around each ending read: can the session lose in-progress work or an unanswered operator exchange, can the tree still change between the read and the verdict, and what happens when the stop or the resume itself fails, including when the session is the caller of `pr-continue`?"
  - "Gate failure mailed during the wait: check `mailed_head` is recorded, deduplicated and cleared correctly across a failed mail, a re-entry, a new commit and `pr-continue`, and that an ungated or failed head can never be pushed or end the review. Check runner crash and memory pressure still halt."
  - "`internal/loomcli/prcontinue.go` admission: the answer key admits orch strands on the operator's word and the run's PR-Review session, and refuses every other agent. Settle whether a pr-reviewer of another run, a look-alike strand name, an untrusted or other-hub caller or a missing or stale record can get through."
  - "Prompt `contracts/stencils/landing/landing-template-pr-review.md` and the `internal/shellguard/rows.go` row: does the session get what it needs to answer comments and fix findings, end only on the operator's word via `pr-continue`, and never approve, merge or push? Check every verb the prompt names exists and is permitted to this role."
  - "Restore fidelity: compare with what `34e64ac2e` removed. The summary says the parent-review ask and the findings send-back are not restored; settle whether the amended answer key licenses each omission and each new behavior, such as the interactive session under autonomy and mail instead of halt."
  - "Un-retiring `pr-reviewer` and `landing.pr_reviewer`: with `agents-pr-reviewer-role-retired` dropped, check hubs and local files that did or did not apply it, any reinterpretation registry or ledger that still refers to it, and whether older records with the removed session keys read safely."
  - "Docs and claims: check orch role files, recipes, `docs/overview.md`, package docs, `PATTERN.md` and `pattern/PATTERN-landing.md`, `internal/loomshed/interruptpolicy.go` and `_lym/landing/summary.md` against the code, including the `## Merge danger` door and blast radius given the record, config and agents-file changes, and whether any rubric or spec text that the retiring commit changed is now stale."
  - "Tests: the diff rewrites `prreview_test.go`, `prreview_wait_test.go` and the seam tests. Check for widened or removed assertions not named by a `Loosened-Check:` trailer, and for new session behaviors left uncovered."
---

Seed focus for round 1, drawn from the amended board entry, the commit list of the review range and the change description.
The later added entry replaces the earlier one's clause that the session merges the pull request itself.
