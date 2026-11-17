# Card 9 — this-machine-names-registry

**Edit:**
- `internal/pairteardown#ThisMachine` -> `func ThisMachine(hubPath string) (fabricengine.MachineNames, error)`
- `internal/pairteardown#PushRun`
- `internal/fabriccli#addWeftVerbs`
- `internal/loomcli#landingDeps`
- `internal/beamcli#hubDobbyDeps`

**Create:**
- `plan:internal/pairteardown#machineNamesOf` -> `func machineNamesOf(here machinename.Identity, registry boardengine.Registry) fabricengine.MachineNames`
- `plan:internal/pairteardown#TestMachineNamesOf` -> `func TestMachineNamesOf(t *testing.T)`

**Uses:**
- `internal/fabriccli/doc.go`
- `internal/beamcli#machineLandings`
- `internal/fabricengine#MachineNames`

**Intent:** Hand `fabricengine`'s pair push, and the landing count, the set of names that read as this machine through the registry (D3, and the transition part's `MachineNames` seam).

`ThisMachine(hubPath)` reads `machinename.ReadIdentity` and the registry of the hub's local Bolt, and returns `machineNamesOf(identity, registry)`:
`Current` is the identity's name; `Former` is the identity's former names plus the current and former names of the registry entry under the identity's id, without the current name and without duplicates.
A machine with no entry gets its file's names alone.
`fabricengine`'s takeover read then reads a takeover entry naming any name of this machine's entry as its own.
Its callers pass the hub: `PushRun` (`prime.HubPath`), the two closures in `addWeftVerbs`, and `landingDeps`.
`hubDobbyDeps`' `Landings` seam hands `machineLandings` a names function over `ThisMachine(prime.HubPath)`, so a landing recorded under any name of this machine's entry counts.

`TestMachineNamesOf`, tier0, covers no entry, an entry holding a former name the file lacks, and a registry entry of another id, which adds nothing.

**ImpactSummary:** One re-signed function with four production callers, all passing the hub path they already hold; the landing count reads the same seam.
