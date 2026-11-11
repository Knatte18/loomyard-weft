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
Its doc comment says it ignores the machine file, unlike `Name`.
`internal/machinename/doc.go` stays true: the package is still the only reader of the hostname.

Test, tier 0: `TestFromThisHost` pins the success path: it equals `FromHostname` of the test's own `os.Hostname()` read.
It reaches no error path, since the test host has a hostname; that path is `nameFromHostname`'s, which `TestResolveNames`' `UnreadableHostnameNamesItsWayForward` row covers.

**Commit:** 2: machinename: export the hostname rule's name of this host
