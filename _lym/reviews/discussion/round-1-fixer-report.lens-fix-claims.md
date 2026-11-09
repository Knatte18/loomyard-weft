```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every claim the fix added or rewrote against the code at HEAD.

Claims that hold:
- `fabricengine.Bolt.Pull` exists (`internal/fabricengine/bolt.go:279`).
- `writeOwners` skips done entries.
- `retireEntry` and `adopt` behave as the new Slack constraint says.
- `standalonestate.UserStateDir` ignores a relative `XDG_STATE_HOME` and refuses a relative `HOME` or `LOCALAPPDATA`.
- `PauseOwnedElsewhere`, or `EndOwnedElsewhere` through the hibernate wake, ends processes on each pause path the record names (`internal/battencli/wire.go:1248`, `internal/beamcli/dobbydeps.go:156`, `internal/hibernatecli/deps.go:133`).
- The dobby gives a full pass at the lift of a covering pause, and its probe lift covers only runs marked owned elsewhere. So "a pause the dobby's probe does not lift" is a real case, because a pass's own check sets no mark.

Findings:

- **LOW** — D6 window bound: "No pause holds every such check: a pause never touches the orch (`internal/tenderengine/doc.go`)". The citation does not support the claim. `tenderengine/doc.go:42` says the pause engine sends Esc only and never touches the orch's pane. It says nothing about whether the orch's owner checks honour a pause. The direct evidence is `internal/orchcli/doc.go:207-210`: `Prepare` pauses each owned_elsewhere entry through `tendercli.PauseOwnedElsewhere` at bring-up, and nothing there is gated by a hub pause.
- **LOW** — D6 first recovery: "a run that takes an owner pause, from the dobby's probe, a pass's own owner check or a hibernate wake". This list of sources is incomplete. The orch's bring-up (`internal/orchcli/doc.go:208`) and loom's pre-push (`internal/loomcli/prepush.go:79`) also call `tendercli.PauseOwnedElsewhere`. The `lym tender resume <slug>` recovery covers both, but as written the list reads as complete.
- **LOW** — D5 lost-name sentence: "restoring the lost name in `names` by hand counts it again". The landing was never counted, because `countLandings` only adds slugs not yet in `CountedLandings`. The restore makes it count, not count again.

Tool calls beyond context:
1. Grepped the `PauseOwnedElsewhere`/`EndOwnedElsewhere` callers and the tender docs, to check which paths end processes and how resume treats owner entries.
2. Grepped the owner-caller rules in `pauserecord`, to check whether the dobby lift and resume apply to owner entries.
3. Read `orchcli/doc.go` on pause and the bring-up owner pause, to test the "a pause never touches the orch" evidence.
