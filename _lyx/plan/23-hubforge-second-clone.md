# Card 23 — hubforge-second-clone

**Create:**
- `internal/hubforge#CloneSecond`

**Uses:**
- `internal/hubforge#CopyHub`

**Intent:**
A second machine on the same remote, for adopt's integration test (D5).
`hubforge.CloneSecond` clones a second hub from `h`'s own bares through `fabriccli.CloneAndWire`, as a second machine working on the same remote.

**ImpactSummary:** hubforge gains one test helper; no existing caller changes.

**Commit:** 23: hubforge: clone a second hub from a hub's own bares
