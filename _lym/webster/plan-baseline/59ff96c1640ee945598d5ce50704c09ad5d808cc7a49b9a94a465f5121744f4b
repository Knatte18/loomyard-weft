# Card 7 — gate-hold-wiring

**Create:**
- `internal/beamcli#gateHoldTopicPrefix`
- `internal/beamcli#gateHoldTags`
- `internal/beamcli#dobbyGateHolders`
- `internal/beamcli#TestDobbyGateHolders`

**Edit:**
- `internal/beamcli#hubDobbyDepsStarting`
- `internal/beamcli#TestRunNoticeTags`
- `internal/beamcli#TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy`
- `contracts/orchrole/orch-role.md`
- `docs/overview.md`

**Uses:**
- `internal/beamcli#runNoticeTags`
- `internal/beamcli#requirePosted`
- `internal/beamcli#pairLocation`
- `internal/dobbyengine#GateHoldDeps`
- `internal/gateslot#Holder`
- `internal/gateslot#Pool.Holders`
- `internal/proc#TreeSample`
- `internal/proc#PIDsWithEnv`
- `internal/gateslot#Pool.Acquire`
- `internal/beamcli/doc.go`

**Intent:**
Wire the gate-hold notice of card 6 into the hub's dobby, and name it to the orch (D6, D7).

`dobbyGateHolders` returns the `Holders` seam: it loads `gateslot.LoadConfig(fabricengine.ConfigDirs(prime))`, reads `hubgeom.GateSlots(prime).Holders()`, and maps each `gateslot.Holder` to a `dobbyengine.GateHolder`, with `TierTimeout` from `cfg.Tiers[holder.Tier].TimeoutMin` minutes, zero for an empty or undeclared tier; a failed load or read is returned wrapped, naming which failed.
`hubDobbyDepsStarting` sets `GateHolds` on the `Deps`: `Holders` from `dobbyGateHolders(prime)`; `TaskWorktree` from `pairLocation(prime, slug)`'s `WorktreePath()`, the path the file watch resolves; `Tree` `proc.TreeSample`; `MarkedPIDs` `proc.PIDsWithEnv`;
`Post` `requirePosted(orchcli.NotifyRunFrom(worktree, geom.Slug, dobbyProgram, gateHoldTags(slug), orchcli.WithRunAutonomy(worktree, slug, line)))`.
`gateHoldTags` returns `creelengine.PostTags{Topic: gateHoldTopicPrefix + slug, Run: slug}`, a topic of its own, so a gate-hold notice and a later run notice of the same run never replace each other.

`contracts/orchrole/orch-role.md`'s Notices section names the gate-hold condition among the `[dobby notice]` conditions: a gate slot a run holds past its bound, naming the holder's process tree and any process outside it that carries its marker; the watch only reports, so the orch decides what to end.

`docs/overview.md`'s dobby paragraph names the gate-hold `[dobby notice]`, a gate slot a run holds past its bound, among what dobby sends the nearest live orch.

Tests:
`TestRunNoticeTags` also asserts `gateHoldTags("pay")` is `{Topic: "dobby-gate-hold/pay", Run: "pay"}`.
`TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy` also posts one line through `deps.GateHolds.Post` for `task1` before the row's run notice, wants it among the posted mails with the same sender and autonomy stamp, and its no-supersede assertion from card 4 then covers the gate-hold notice and the later run notice standing unread together.
Its doc comment and its file comment name the gate-hold notice beside the run notice and the watch line.
`TestDobbyGateHolders` (tier 1) runs over a `hubforge` hub whose gate config declares `tier1` with `timeout_min: 35`.
It holds a slot of `hubgeom.GateSlots(prime)` with `Tier: "tier1"`, one with an empty tier and one with an undeclared `tier9`, and wants `dobbyGateHolders(prime)()` to answer `TierTimeout` 35 minutes for the first and zero for the other two.
A second row breaks the hub's gate config and wants an error naming the gate config load.

**ImpactSummary:** One seam struct wired in the hub's dobby, one tag helper, and one Notices bullet; `beamcli` gains a `gateslot` import and no existing seam changes.

**Commit:** 7: Post the gate-hold notice from the hub's dobby and name it to the orch
