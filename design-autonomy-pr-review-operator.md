# Autonomy: let PR review stay with the operator for one beam

`autonomy-pr-review-operator` · Note · enhancement, landing, config

Imported from [issue #217](https://github.com/Knatte18/loomyard/issues/217).

`beam.yaml: autonomy` is one switch for both halts and PR-Review requests. A hub wants the orch to settle halts on the prime autonomously while pull requests into the prime's branch (main) still wait for the operator's approval. Today that needs autonomy off, which also sends every halt to the operator, or autonomy on with the orch holding PR-Review by hand. Proposal: a separate key, such as `autonomy_pr_review: false`, so PR-Review requests on that beam or the prime go to the operator while halts stay autonomous.

## Issue comment, 2026-11-24

Seen again, with a merge into the prime's branch:

- The prime runs with `autonomy: true`, and the hub wants pull requests into main to wait for a human approval.
- The task `import-project-names` reached PR-Review; the parent answered the parent-review ask `--satisfied` and told the PR-Review agent by creel that landing waits on the operator's approval.
- `lym loom pr-continue` ran anyway about a minute later, and Finalize squash-merged the pull request into main once its gate passed, with no human approval on GitHub.

A message to the PR-Review agent is no way to hold a landing under autonomy: the satisfied answer alone ends the review.

Expected: a setting, per beam and for the prime, that keeps autonomy for halts but makes PR-Review wait for a human approval of the pull request (for example a GitHub approving review) before `pr-continue` or Finalize may merge.