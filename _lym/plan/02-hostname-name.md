# Card 2 — hostname-name

**Create:**
- `plan:internal/machinename#HostnameName` -> `func HostnameName() (string, error)`

**Uses:**
- `internal/machinename/doc.go`
- `internal/machinename#TestResolveNames`

**Intent:** Export the hostname rule's name, so `lym machine name` can take it as one of this machine's names (D5) while `internal/machinename` stays the only production reader of the hostname.
`HostnameName` returns `nameFromHostname(os.Hostname)`, the name the hostname rule gives this machine whatever the machine file holds, and its error names the way forward `nameFromHostname` already names.
This card departs from the decision record's Scope list, which names only `internal/boardengine`, `internal/machinecli` and `cmd/lym`.
None of the functions the record's Takes line names returns the hostname rule's name while a machine file exists.
Reading `os.Hostname` in `internal/machinecli` would break `PATTERN-owner-machine`'s rule that `internal/machinename` is the only production reader of the hostname.
The overview's Shared Decisions record the departure.
`TestResolveNames` already covers `nameFromHostname`'s rule and its error, so the card adds no test of its own:
the exported function reads the real hostname, which no test controls, and `TestSoleSource_NoOtherProductionFileReadsTheHostname` keeps every other package off `os.Hostname`.
