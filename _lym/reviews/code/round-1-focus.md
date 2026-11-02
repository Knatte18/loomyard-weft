---
round: 1
focus:
  - "internal/adoptcli/adopt.go, the `rootGoing` gate around `deployStale`: settle whether it is true exactly when root comes to this machine and the prime's owner write will stand, or also when a stop holds root back, when the prime was already owned here, or on a path that returns before the owner write. The board entry asks for a deploy when `lym adopt root` runs with the prime's `main` ahead of the installed build."
  - "internal/adoptcli/deploy.go `installedBehind` and `readDeployState`: settle whether `buildvcs.Identity.Revision` and `fab.HeadSHA()` are the same length and form, since `installed.Revision == head` decides between no deploy and a needless deploy on every adopt. Settle also what a build marked `Modified` and a build of the head itself should do, and whether the not-behind reads (commit absent, not an ancestor) can leave the incident's stale install undeployed."
  - "internal/adoptcli/adopt.go failure path after `deployStale` errors: settle whether `finish()` and the returned `result` leave the holds, the mutation record and the `Partial` flag consistent with the refusal text (`no owner was recorded and the holds stay`), and whether the deploy log path and `deploy` envelope field survive to the caller on the error return."
  - "internal/adoptcli/deploy.go `runAdoptDeploy` and `readDeployState`: settle whether `testing.Testing()` in non-test production code, the `subjectDir` scratch location, and the deploy running in `prime.WorktreePath()` after the reconcile match the repository's constraints document and the way the Land row runs the same command. Check the new `battencli` and `battenshed` exports, and the `adoptcli` imports of them, against any package-layering rule."
  - "Test coverage of the new behavior: `readDeployState`, `runAdoptDeploy`, the `hubDepsAs` wiring and `fabricengine.Fabric.CommitExists` have no test of their own in the diff. Name each uncovered behavior, and check the `TestAdopt` rows and the `TestAdoptFields` row against existing tests for duplicated coverage."
  - "internal/adoptcli/cli.go long help and doc.go: settle whether the help's step list, the dropped envelope field list, and the doc's `# The deploy` section agree with the code's order (hold, deploy, owner write, beam steps, checks) and field names. Check the summary's claim that the help stays under the 2000-byte cap."
---

Range reviewed against the decision record's `## Goal`, the only answer key, which has no plan.
The change description is a claim to check, not the measuring stick.
