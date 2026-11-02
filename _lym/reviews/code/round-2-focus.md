---
round: 2
focus:
  - "internal/adoptcli/adopt.go, the deploy gate over rootGoing, and the TestAdopt rows in internal/adoptcli/adopt_test.go that cover it: does a test fail when the held-root clause is dropped, and is the deploy-before-owner-write order pinned for a prime already owned by this machine under adopt root?"
  - "internal/adoptcli/deploy.go readDeployState and its tests in internal/adoptcli/deploy_test.go: are the empty-deploy short-circuit, the deploy_branches miss, the build-info read error and the CommitExists and IsAncestor wiring into installedBehind each pinned by a row, and do the tests stay hermetic under PATTERN-test-isolation?"
  - "internal/adoptcli/deploy.go runAdoptDeploy and the function mapping a battenshed.DeployResult and error to the refusal: does every outcome (start error, timeout, non-zero exit, zero exit) yield a message that names the way forward per PATTERN-refusal-way-forward, and is each outcome tested?"
  - "internal/adoptcli/adopt_test.go failed-deploy row: on the refusal return, are Partial and the mutation record from finish() asserted, and do the holds stand with no owner write?"
  - "pattern/PATTERN-cli-cobra.md adoptcli line and docs/overview.md adopt bullet: do both describe the deploy step and the packages adoptcli now imports, matching the code?"
---

Focus targets the code and tests that carry the deploy step; the review range is unchanged.
