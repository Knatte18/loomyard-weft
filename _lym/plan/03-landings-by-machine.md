# Card 3 — landings-by-machine

**Edit:**
- `internal/battencli#DobbyLandings` -> `func DobbyLandings(prime *lymcwd.Location, geom dobbyengine.Geometry, parents *DobbyParents, machine string) ([]string, error)`
- `internal/beamcli#hubDobbyDeps`
- `internal/dobbyengine#Deps`
- `internal/dobbyengine/doc.go`

**Create:**
- `plan:internal/beamcli#TestDobbyLandings_CountsOnlyThisMachine` -> `func TestDobbyLandings_CountsOnlyThisMachine(t *testing.T)`
- `plan:internal/beamcli#machineLandings` -> `func machineLandings(machine func() (string, error), read func(machine string) ([]string, error)) func() ([]string, error)`
- `plan:internal/beamcli#TestMachineLandings` -> `func TestMachineLandings(t *testing.T)`

**Extends:**
- `internal/beamcli/doc.go`

**Uses:**
- `internal/beamcli#seedBattenRun`
- `internal/shedengine#HistoryEntry`

**Intent:**
Decision 8: a dobby counts only the landings its own machine made.

`DobbyLandings` gains the `machine` parameter and returns a run only when its history holds an entry whose `Producer` is Land, `Outcome` is Done and `Machine` equals `machine`.
An entry with an empty `Machine`, written before card 1, is never counted, and neither is one naming another machine.
Its doc comment says so.

`hubDobbyDeps` wires `Landings` to `machineLandings(machinename.Name, …)`, which reads the machine's name on each call and passes it to `DobbyLandings`.
A name read that fails, or an empty name, returns `this machine's name is unknown, so its landings cannot be told apart: <error>; way forward: give the machine a hostname`, which the tick reports as a failed landing read (`read the landings: …`) through the loop's existing tick-failure episode.
`TestMachineLandings` (tier0) has three rows: a name `m1` reaches `read` with `m1` and returns its slugs; a failing name read returns an error naming the read's error and `give the machine a hostname`, and never calls `read`; an empty name does the same.

`internal/dobbyengine#Deps`' `Landings` field doc and `internal/dobbyengine/doc.go`'s board-tidy bullet, which say every Land Done is counted, now say only the landings whose Land Done names this machine are.

`TestDobbyLandings_CountsOnlyThisMachine` (tier1, beside `TestDobbyRuns_PairAtItsOwnAnchor`, over a `hubforge` hub) seeds three batten runs of the prime with `seedBattenRun`, then overwrites each status file with a history holding a Land Done entry naming `m1`, one naming `m2`, and one naming none; `DobbyLandings(prime, dobbyengine.Geometry{}, battencli.NewDobbyParents(prime), "m1")` returns exactly the first slug.

`internal/beamcli/doc.go`'s seam list gains, at the `battencli.DobbyLandings` bullet, that the landings are read for this machine's name only.

**ImpactSummary:** One re-signed exported reader whose only caller is the dobby's Landings seam in `hubDobbyDeps`, now wrapped by a small name-reading helper; two doc texts follow.

**Commit:** 3: Count only the landings this machine made
