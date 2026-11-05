# Relevant files — pawl-orch-landing

- `contracts/recipes/pawl-recipe.yaml` — the rows D1 drops and the hold-boundary comment
- `internal/pawlengine/beamawait.go` `BeamAwait.Call`, `awaitRun.accept`, `awaitRun.refuse`, `awaitRun.answerReopen`, `BeamAwait.interruptedLanding` — the accept's hold write, the reopen handling D10 removes and the crash check D2 renames
- `internal/pawlengine/doc.go` — package doc that names Parent-Align and the reopen request
- `internal/beamcli/land.go` `reopen`, `land`, `markReady`, `judgeLandEntry`, `judgeLandRun` — the ready verb D3 extends and the reopen verb D10 removes
- `internal/beamcli/mergeparent.go` `mergeParent` — the merge-in task D3's refusal names
- `internal/boardengine/runlock.go` `RunHolds` — the hold-free rows D2 keeps
- `internal/landingshed/beamdescribe.go` `NewBeamDescribe`, `beamDescription`, `beamCommitEntry` — the description writer D5 moves behind a verb
- `internal/landingshed/finalize_pr.go` `pullRequestLanding`, `mergeOnGitHub`, `openAtOnce` — Finalize's pull-request landing D8 and D9 change
- `internal/landingshed/pullrequestmode.go` `ResolvePullRequest`, `resolveWithParentRule`, `checkOrigin` — the resolver D8 extends and D6's origin check
- `internal/loomshed/interruptpolicy.go` `InterruptPolicies` — the table D11 adds the pawl rows to
- `internal/sheddriver/driverules.go` — re-steps an interrupted stop under the re-invoke policy up to its repair cap, which D11 relies on
- `internal/arbiterstop/arbiterstop.go` — the arbiter's eligibility for interrupted stops, which D11 extends to pawl
- `internal/fabricengine/pushlockholder.go` `describePushLockHolder` — the busy report behind the #47 open risk
