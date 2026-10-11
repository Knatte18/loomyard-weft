# Card 15 — battencli maps the child's driver-busy refusal to batten's sentinel

**Create:**
- `internal/battencli#childRefusalKind`

**Edit:**
- `internal/battencli#childSpawnError`
- `internal/battencli#TestChildSpawnError`

**Delete:**
- `internal/battencli#childNotParked`

**Uses:**
- `internal/battenshed#ErrChildDriverBusy`
- `internal/shedrun#StartDriverBusyKind`
- `internal/battenshed#ErrChildNotParked`
- `internal/shedrun#StartNotParkedKind`

**Intent:**
Carry the child's `driver_busy` refusal across the process boundary to batten's retry (decisions D8 and D9), the way the not-parked refusal already is.

`childRefusalKind` generalises `childNotParked`: it tries each output line as a JSON envelope and returns the `kind` of the first line with `ok` false and a non-empty kind, or "" when none carries one.
`childSpawnError` maps that kind: `shedrun.StartNotParkedKind` wraps `battenshed.ErrChildNotParked` as today, and `shedrun.StartDriverBusyKind` wraps `battenshed.ErrChildDriverBusy`, each beside `runErr` and the truncated output; any other kind keeps the plain wrap.
`childNotParked`, whose only caller was `childSpawnError`, is deleted, and the comment on `childSpawnError` names both mappings.

Test: `TestChildSpawnError` gains rows: an output whose envelope carries `driver_busy` wraps `ErrChildDriverBusy` and not `ErrChildNotParked`; a `driver_not_parked` envelope still wraps `ErrChildNotParked`; an envelope with `ok` true or another kind wraps neither.

**ImpactSummary:** One unexported helper replaced in battencli's wire file; its sole caller is on this card.
