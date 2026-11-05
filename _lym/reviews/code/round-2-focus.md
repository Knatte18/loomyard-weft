---
round: 2
focus:
  - "internal/landingshed/finalize_pr.go `pullRequestLanding`: settle whether the beam-without-pull-request branch that takes a run's scratch record forces `Open` and `MergeAtOnce` only when GitHub reports that record's pull request open, and what a closed or merged scratch pull request does under `on`, `off` and `required` parents."
  - "internal/landingshed/finalize_pr.go `recordedBeamLanding` with internal/loomcli/landingdeps.go `CommitsMissing` and internal/beamcli/deps.go `MergedLacks`: settle whether Finalize and `lym beam ready` judge a merged pull request as lacking beam commits by one rule, so a beam past the merged head only by a parent merge-in or records-only commit is not read as lacking."
  - "internal/beamcli/deps.go `ParentUnmerged` and internal/beamcli/deps_integration_test.go: settle whether the branch that fetches the parent's upstream under the remote-refs lock and judges its tip fails a test row when it breaks, with the push skip unset."
---

Focus names the three gating areas of the round-1 review; the remaining findings are LOW and ride along with ordinary review.
