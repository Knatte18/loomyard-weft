# Card 1 — machinename-hostname-name

**Create:**
- `plan:internal/machinename#HostnameName` -> `func HostnameName() (string, error)`

**Edit:**
- `internal/machinename#TestFromHostname`

**Uses:**
- `internal/machinename#nameFromHostname`
- `internal/machinename#FromHostname`
- `internal/machinename/doc.go`

**Intent:** Export the hostname rule's name on its own, per the decision record's added-after-Discussion entry "machinename exports HostnameName for the hostname rule".
`HostnameName` returns `nameFromHostname(os.Hostname)`: the hostname cut at its first dot, lowercased and trimmed, or the existing error naming its way forward when the hostname does not read or yields no name.
It reads no machine file, so `lym machine name` (card 3) gets the hostname rule's name over a bad file too.
`os.Hostname` stays read inside `internal/machinename` alone, so `TestSoleSource_NoOtherProductionFileReadsTheHostname` keeps holding.
The doc comment says it ignores the machine file, unlike `Name` and `Names`.

`TestFromHostname` gains one assertion, after its table: `HostnameName()` equals `FromHostname` applied to the test's own `os.Hostname()` result, error for error.
The test file is not production code, so the sole-source scan does not read it.

**ImpactSummary:** One new exported function in a leaf package and one added assertion; no existing caller changes.
