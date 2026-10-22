# Card 13 — tender-owner-pause

**Create:**
- `plan:internal/tendercli#PauseOwnedElsewhere` -> `func PauseOwnedElsewhere(location *lyxcwd.Location, slug string) error`

**Uses:**
- `internal/tenderengine#Pause`
- `plan:internal/pauserecord#OwnerCaller`

**Intent:**
The one owner pause every caller uses (D6).
`tendercli.PauseOwnedElsewhere` builds tender's hub deps with `HubDeps` and pauses the slug through `tenderengine.Pause` with a ten second grace and `Caller` `pauserecord.OwnerCaller`, so tender interrupts the child's running turns and a slug entry already standing adds none.

**ImpactSummary:** tendercli gains one exported function; no existing caller changes.

**Commit:** 13: tender: one owner pause of a slug
