# Card 2 — hostname-name

**Create:**
- `plan:internal/machinename#HostnameName` -> `func HostnameName() (string, error)`
- `plan:internal/machinename#TestHostnameName` -> `func TestHostnameName(t *testing.T)`

**Extends:**
- `internal/machinename/doc.go`

**Uses:**
- `internal/machinename#nameFromHostname`
- `internal/machinename#FromHostname`

**Intent:** Export the hostname rule's name on its own, per the decision record's added entry, so `lym machine name` can count it among this machine's names (D5) while `internal/machinename` stays the only production reader of the hostname.

`HostnameName` returns `nameFromHostname(os.Hostname)`: the hostname cut at its first dot and lowercased, or the error naming its way forward.
It reads no machine file, so it answers the same over a bad file.
`doc.go` gains a sentence: `HostnameName` gives the hostname rule's name regardless of the machine file, for the verb that records it among the machine's names.

`TestHostnameName` (tier0) asserts that `HostnameName()` equals `FromHostname` applied to the test's own `os.Hostname()` read, value and error alike; the sole-source scan reads production files only, so the test's read passes it.

**ImpactSummary:** Adds one exported function and a doc sentence to internal/machinename; no existing caller changes.
