# Card 2 — machinename-hostname-name

**Create:**
- `plan:internal/machinename#HostnameName` -> `func HostnameName() (string, error)`

**Edit:**
- `internal/machinename#TestFromHostname`

**Extends:**
- `internal/machinename/doc.go`

**Uses:**
- `internal/machinename#nameFromHostname`
- `internal/machinename#FromHostname`

**Intent:**
Export the hostname rule's name, as the parent's added decision records, so `lym machine name` can count it among this machine's names without reading the hostname outside `internal/machinename`.

`HostnameName` returns `nameFromHostname(os.Hostname)`: the hostname cut at its first dot and lowercased, or the error naming the way forward when the hostname does not read or yields no name.
It reads the hostname on every call, whatever the machine file holds.

`TestFromHostname` (tier0, its file's existing tier) gains an assertion after its table: `HostnameName()` equals `FromHostname` applied to the test's own `os.Hostname()` read, error for error.
A test file is outside the sole-source scan, which reads production files alone.

`internal/machinename/doc.go` gains one line naming `HostnameName` as the hostname rule's name for a caller that needs it beside the file's names.

**ImpactSummary:** one new export in a leaf package; no existing caller changes.

**Commit:** 2: machinename exports the hostname rule's name
