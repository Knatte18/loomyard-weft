# Card 2 — hostname-name

**Create:**
- `plan:internal/machinename#FromThisHost` -> `func FromThisHost() (string, error)`
- `plan:internal/machinename#TestFromThisHost` -> `func TestFromThisHost(t *testing.T)`

**Uses:**
- `internal/machinename#nameFromHostname`
- `internal/machinename#FromHostname`
- `internal/machinename/doc.go`

**Intent:**
`machinename.FromThisHost` returns the hostname rule's name of this host, `nameFromHostname(os.Hostname)`, whatever the machine file holds.
`lym machine name` (cards 3 and 4) needs that name beside the file's names (D5), and `internal/machinename` is the only production reader of the hostname (`PATTERN-owner-machine`'s sole-source scan, `TestSoleSource_NoOtherProductionFileReadsTheHostname`), so `internal/machinecli` takes it through this export rather than reading the hostname itself.
Its error is `nameFromHostname`'s, which names its way forward.
Its doc comment stands alone and names no other symbol: "FromThisHost returns the hostname rule's name of this host and never reads the machine file; it is for the command that shows and sets the machine file, which needs that name beside the file's names".
Bound: `FromThisHost` serves `lym machine name` alone, its show and its set, which use it as this machine's name only when no machine file exists;
no other code takes an owner, address or channel name from it, since `Name` and `Names` are the one source of this machine's name, and a machine file that yields no valid name makes them fail rather than fall back to the hostname.
`internal/machinename/doc.go` stays true: the package is still the only reader of the hostname.

Test, tier 0: `TestFromThisHost` pins the success path: it equals `FromHostname` of the test's own `os.Hostname()` read.
It reaches no error path, since the test host has a hostname; that path is `nameFromHostname`'s, which `TestResolveNames`' `UnreadableHostnameNamesItsWayForward` row covers.

**Commit:** 2: machinename: export the hostname rule's name of this host
