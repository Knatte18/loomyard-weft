# Card 16 — prime-pull-owed-tips

**Edit:**
- `internal/fabricengine#Fabric.Pull`
- `internal/battencli#landDeps`
- `internal/battenshed#landProducer.pullAndDeploy`
- `internal/battenshed#TestLand_Dispositions`

**Create:**
- `plan:internal/fabricengine#primeAheadByOwedTips` -> `func primeAheadByOwedTips(l *lymcwd.Location) (bool, error)`
- `plan:internal/battencli#TestLandDeps_PullWithAnOwedLandingPush` -> `func TestLandDeps_PullWithAnOwedLandingPush(t *testing.T)`

**Uses:**
- `internal/fabricengine/doc.go`
- `internal/fabricengine#PullHub`
- `internal/fabricengine#pullPairSide`
- `internal/battencli#pullStopOutcome`
- `internal/pushledger#OwedTips`
- `internal/pushledger#LandingKey`
- `internal/fabricengine#AheadByOnly`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`

**Intent:** D5: every pull of the prime, batten's Land row among them, leaves the owed landing tips alone, and Land waits out an unreachable origin.

`primeAheadByOwedTips` reports whether the prime's local code branch is ahead of its tracking ref by only tips the landing target records as owed:
`fabricengine.AheadByOnly` over `pushledger.OwedTips(hub, LandingKey)`, so every first-parent commit past origin must be an owed tip, and an unrecorded commit below one is judged as today.
`Fabric.Pull` on the prime reads such a branch as level and moves no code side over it; any other ahead commit is judged as today.
`PullHub` pulls the prime through `pullPairSide` and `Fabric.Pull`, so the hub pull keeps the rule, and so does every other caller of the prime's pull.

Batten's Land row, which pulls the prime right after Finalize, reads an unreachable origin as a `git-transport` stop, whether or not a landing push is owed:
`landDeps`'s `Pull` seam marks a pull error whose cause is a git transport failure with `shedengine.MarkTransient(shedengine.TransientGitTransport, …)`,
and `landProducer.pullAndDeploy` returns an error carrying a transient class as the error, not as a Stuck, so the Go driver waits it out (D6).
Any other pull failure stays Stuck as today, and `pullStopOutcome`'s records-side stop keeps its notice.
The `LandDeps` struct keeps its fields, so the seam tests listed under Uses stay as they are.

Tests.
`TestLand_Dispositions`, tier0, adding under a second to one tier0 run, gains a row: a pull error marked `git-transport` returns that error and no Stuck.
`TestLandDeps_PullWithAnOwedLandingPush`, tier1, over a hub fixture whose prime code branch holds one landing tip origin lacks, recorded in the ledger:
with origin reachable the pull passes and the local branch keeps the tip, both with the landing push owed and after it landed;
a prime with an unrecorded commit below the owed tip is judged as an ahead branch, as today;
with origin unreachable the pull's error carries the `git-transport` class, both with the push owed and with it landed.

**ImpactSummary:** Narrows the prime pull's ahead judgement to the owed landing tips and turns Land's unreachable pull into a network stop; no signature changes.
