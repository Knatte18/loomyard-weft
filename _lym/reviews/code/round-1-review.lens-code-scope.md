```yaml
lens: code-scope
outcome: reported
```

**Decisions (the answer key is a decision record alone with no `Added after Discussion` entries; goal points of the board entry's `## Goal`):**
- G1 (Decision): the description is updated at every member landing, rewritten whole, never appended to.
- G2 (Design 1): on a member landing under a recorded PR, lym starts a describer agent on the beam; it reads `summary.md`, the beam's commits and every member PR, and rewrites the end state with every member PR listed.
- G3 (Design 2): `lym beam describe` records it and updates the PR, as today.
- G4 (Design 3): no orch is asked; the landed line drops its follow-up.
- G5 (Design 4): a failed rewrite leaves the old description standing, retried at the next landing; hand edit-and-record still works.
- G6 (Design 5): where it runs and how the describer role is reused is for the run to settle.

**Per change → decision**
- C1 Dobby rewrite step — `internal/dobbyengine/describe.go` (describeStep, spawnDescribe, settleDescribe, endDescribe, draftFindings, stopDescribeStrand), `state.go` `LandedSinceDescribe`/`DescribeState`, `tidy.go` countLandings append, `loop.go` tick call, `Deps.BeamDescribe`, `geometry.go` `DescribeDir`/`DescribeDirName`, `hubgeom/dobbygeom.go` wiring → G1, G2, G5, G6 (dobby after the landing).
- C2 Describer prompt — `contracts/stencils/beam/beam-template-describe.md`, `stencils.go` registry + roleOpeningStencils, `BeamDescribePrompt`/`BeamDescribeSpec` (role `beam-describe`, RoleDescriber, Describe segment) → G1 (rewritten whole), G2 (end state, every member PR), G6 (describer role reuse).
- C3 Record-from-draft — `rewriteDescription`, `judgeDescribable`/`recordDescription` extraction, `RecordDescription` `from` param (`land.go`, `deps.go` choosing `ReplaceBeamDescription`), `landingshed.ReplaceBeamDescription` → G3, G5 (validate-before-write, restore on failed commit, hand path kept with `from == ""`).
- C4 Hub wiring — `beamDescribeDeps`, `beamDescribeFacts`, `describeGate` + constants, `descriptionFindings`, `hubDobbyDeps` hookup → G2 (facts: description, branches, changes), G6 (landing.yaml `describe`/`describe_timeout_min`, Describe row's description gate).
- C5 Renames/generalization — `runnerTidyShuttle`→`runnerAgentShuttle` with optional gate, `TidyShuttle`→`AgentShuttle`, `resolveTidyModel`→`resolveModel(dirs, spec)` → forced by C4 (describer shares the adapter, needs a gate and a second model spec).
- C6 Landing-notice follow-up removal — `landnotice.go` `pullRequestFollowUp`, `rememberRuns` `decides`/`followUp` params, `landNoticeLine` `followUp` param, `landNoticePullRequest` episode replaced by `describePullRequest` → G4.
- C7 `landnotice_test.go` follow-up rows replaced by one no-follow-up row → forced by C6 (removed behavior, not a loosened check).
- C8 `landingshed.RenderBeamChanges` exported, `beamDraft` reusing it → forced by C4 (`beamDescribeFacts.Changes`).
- C9 `describeVerb` const and `describeFailure` signature → forced by C3 extraction.
- C10 `shuttleparity` producer `buildBeamDescribe` → forced by C2 under PATTERN-engine-parity (new role spec must be in the parity manifest).
- C11 Docs — `contracts/orchrole/role-beams.md`, `docs/overview.md`, `internal/dobbyengine/doc.go`, `internal/beamcli/doc.go`, `describe.go`/`dobbydeps.go` file headers, `loop.go` comments, `landingshed/beamdescribe.go` header → forced by C1/C3/C6 (stale text otherwise).
- C12 Tests — `describe_test.go` (both packages), `dobbydeps_test.go`, `deps_integration_test.go` additions, `dobbydeps_integration_test.go` assertions, `land_test.go` `recordedFrom`, `TestReplaceBeamDescription` → forced by C1–C4.

**Per decision → changes**
- G1: C1, C2
- G2: C1, C2, C4
- G3: C3
- G4: C6 (+C7, C11)
- G5: C1 (settleDescribe/endDescribe keep description standing, post why, later landing rewrites), C3 (validate-before-write, restore on failed commit, hand path)
- G6: C1, C2, C4

**Findings**

None. Every change traces to a goal point or is forced by one, and every goal point has a change building it.

(Considered and not filed: G2 says the agent "reads … every member pull request"; the prompt lists each member PR by number from its merge commit and has the agent read each member's commit message and the beam's diff rather than fetching PR bodies from GitHub. PATTERN-github-auth confines GitHub calls to `internal/githubclient`, so an agent shelling `gh` would breach a constraint; the merge commit's `(#n)` subject plus the commit list is a sound reading of "reads every member pull request", so this is not a scope gap.)

Tool calls: none — the inherited context held everything this lens needed.
