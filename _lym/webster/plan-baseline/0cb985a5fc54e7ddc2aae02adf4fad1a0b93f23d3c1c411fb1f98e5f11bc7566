# Card 2 — machine-identity

**Edit:**
- `internal/machinename#File`
- `internal/machinename/doc.go`

**Create:**
- `plan:internal/machinename#Identity` -> `type Identity struct`
- `plan:internal/machinename#ReadIdentity` -> `func ReadIdentity() (Identity, error)`
- `plan:internal/machinename#resolveIdentity` -> `func resolveIdentity(locate fileLocator, hostname func() (string, error)) (Identity, error)`
- `plan:internal/machinename#NewID` -> `func NewID() (string, error)`
- `plan:internal/machinename#TestResolveIdentity` -> `func TestResolveIdentity(t *testing.T)`

**Uses:**
- `internal/machinename#resolveNames`
- `internal/machinename#TestResolveNames`

**Intent:** Give the machine file a random machine id and a forgotten-names list, and give callers one read of this machine's whole identity (D3).

`machinename.File` gains `ID string` (`json:"id,omitempty"`) and `Forgotten []string` (`json:"forgotten,omitempty"`).
`ID` is the random id lym writes at the machine's first registration; `Forgotten` holds the former names `lym machine name --forget-former` dropped, which only the copied-file check reads.

`Identity` carries `ID`, `Name` (the current name), `Former` and `Forgotten`.
`ReadIdentity` resolves it through `resolveIdentity(machineFile, os.Hostname)`:
the name and former names exactly as `resolveNames` resolves them (it calls `resolveNames`), and `ID` and `Forgotten` from a decoded file.
With no file, or no directory to hold one, `ID` and `Forgotten` are empty, so the machine reads as unregistered.
An error is what `resolveNames` returns for the same file.

`NewID` returns 32 lower-case hex characters from 16 bytes of `crypto/rand`; its error is the read's, naming no way forward beyond a retry since `crypto/rand` fails only on a broken system.
The package still imports only the standard library and `internal/standalonestate`; it writes no file (the writers are `boardengine`'s registration, card 4, and `lym machine name`, card 10).

`TestResolveIdentity`, tier0, is a table over a temporary machine file: a file with an id and forgotten names; a file with neither; no file (hostname rule, empty id); no directory (empty id); an unreadable file (the `resolveNames` error).
`doc.go` adds the id, the forgotten names and `ReadIdentity`, and names the id's two writers.

**ImpactSummary:** Two optional fields on File and a new reader; every existing caller of Name and Names reads the same values, and older files decode with an empty id.
