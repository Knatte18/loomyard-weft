# Card 2 — The hostname rule's name from machinename

**Create:**
- `plan:internal/machinename#HostnameName` -> `func HostnameName() (string, error)`
- `plan:internal/machinename#TestHostnameName` -> `func TestHostnameName(t *testing.T)`

**Extends:**
- `internal/machinename/doc.go`

**Uses:**
- `internal/machinename#nameFromHostname`
- `internal/machinename#FromHostname`

**Intent:** Export the hostname rule's name of this machine, so `lym machine name` takes it without reading the hostname itself.

D5 has the verb take the hostname rule's name beside the current and former names, even when a machine file exists or is bad.
`Names` gives that name only when no file exists, and `PATTERN-owner-machine`'s sole-source scan,
`TestSoleSource_NoOtherProductionFileReadsTheHostname`, lets no production file outside `internal/machinename` read the hostname.
This card is a departure from the decision record's Scope, which does not list `internal/machinename`; the overview's Shared Decisions states it.

`HostnameName` returns `nameFromHostname(os.Hostname)`: the hostname reduced by the hostname rule, never the machine file's name.
Its error is `nameFromHostname`'s, which already names the way forward.
Its doc comment says it is not this machine's name when a machine file exists, and that owner checks take `Name`.

`TestHostnameName` is tier0 and parallel.
It asserts that `HostnameName()` equals `FromHostname` of the hostname the test reads with `os.Hostname`, error for error and name for name.
A test file may read the hostname, since the sole-source scan walks production files only.

`internal/machinename/doc.go` gains one sentence: `HostnameName` gives the hostname rule's name alone, for the rename verb's former-name set.

What this lets through: a caller can read the hostname rule's name while a machine file names the machine otherwise.
Bound: it returns a name only, the package stays the sole reader of the hostname, and no owner check takes it.

**ImpactSummary:** Adds one exported function and its test to `internal/machinename`; `Name`, `Names`, `FromHostname` and every existing caller are unchanged.

**Commit:** 2: machinename: export the hostname rule's name
