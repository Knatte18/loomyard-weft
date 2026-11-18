```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: check that every refusal, way forward, step and criterion the fix introduced can be carried out and makes progress.

Ways forward checked:
- D2's Preflight refusal of `packages` with no test of the `harden-live` tiers names two ways forward, the `run_settings` write of `packages` and `lym config gate --set` of `harden-live`. Both writes work from the refused state.
- D9's seed refusal of a Guard key in `run_settings` goes through `HubOnlyWayForward`. It can be carried out: `configreg.PrepareRunSettings` calls `refuseHubOnlyKeys` and ends the refusal with the spec's text.
- D6's escalation of a red auto-accept round goes to the parent "as a capped campaign does", through the Bouncer's existing parent notice and decision verbs. It can be reached.
- D4's live run that starts no test binary reads red. It reaches the existing `live_attempts` path and then a capped end with a parent decision, so it ends somewhere that makes progress.
- The `--copies` criterion (n equal to the pool's slot count completes) can be reached under the stated lease rule.
- D5's pairteardown route spawns `lym harden sandbox clear`. See the finding below.

Findings:

- **MEDIUM**: D5, the new sentence "`internal/pairteardown` also runs it before it removes a worktree whose sandbox root exists, by spawning `lym harden sandbox clear` in that worktree".
  - Summary: the step runs `clear`, but D5 defines `clear` as narrower than the reap. So the operator-removal route that F19 meant to fix still leaves the sandbox hubs' recorded processes (dobbies) and unrecorded processes alive.
  - Evidence: D5 says "`lym harden sandbox clear` stops each hub's reed server and removes the root's hubs". Only the reap "ends each sandbox hub's recorded processes through `runproc.End` … stops each listed reed server, kills by the `/proc` scan only the processes no record names". F19's summary names "reed servers and dobbies" as what survives.
  - Either `clear` runs the full reap, or the sentence and the new criterion "clear its sandbox root" should say that `clear` does the reap's process ends too. Otherwise the step can be carried out but does not reach the state it claims: nothing left behind.

Further tool calls:
- Read `internal/configreg/runsettings.go` (grep for `HubOnlyWayForward`), because my context had only `doc.go`'s summary. I needed to confirm that the run-settings refusal carries the text as its way forward.
