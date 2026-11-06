# Card 5 — hubs-verb

**Create:**
- `plan:internal/creelcli#hubsCmd` -> `func hubsCmd() *cobra.Command`
- `plan:internal/creelcli#rig.addHub` -> `func (r *rig) addHub(t *testing.T, shortname string) *rig`

**Edit:**
- `internal/creelcli#Command`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/hubregistry#Live`
- `internal/creelconfig#LoadConfig`
- `internal/hubgeom#CreelGeometry`
- `internal/creelcli#clientSeam`

**Intent:** Add `lym creel hubs` (D8), which lists the machine record's hubs that pass D1's check.

The verb runs through `runVerb`, so a standalone directory is refused like every creel verb, and its audience is `clihelp.AudienceRole`, as `read` and `inbox` are, since the orch role names it.
It reads `hubregistry.Live` from `seam.registryDir()` and prints `{"hubs":[{"shortname":…,"hub_path":…,"daemon_answers":…,"trusted":…}]}`, an empty list when the record holds none.
`daemon_answers` dials the hub's socket from `hubgeom.CreelGeometry`, closes the connection and starts no daemon, so the verb never starts a hub's daemon and never writes the record.
`trusted` is whether this hub's `creelconfig.LoadConfig` lists the shortname.
A corrupt record refuses with `hubregistry`'s way forward, and an invalid `creel.yaml` refuses with `configengine`'s error and the way forward of correcting it with `lym config creel`; a read of either that fails otherwise is propagated raw.
A failing `seam.registryDir` is propagated raw too, so a missing home reads as `standalonestate`'s refusal, which names `XDG_STATE_HOME` and `HOME`.
The verb can now show the paths of the operator's hubs to any strand of the user, and nothing more: it is read-only, and the record is a file that strand's user can read already.

`TestCreelCLI` rows, tier1, with `seam.registryDir` from `newRig`: `hubs` joins the verbs of the standalone-refusal row; `hubs` with an empty record prints an empty list; a recorded hub with a running daemon shows `daemon_answers: true`, one with none `false` and no daemon started; a hub named in `trusted_hubs` shows `trusted: true`, another `false`; a removed hub is not listed; a corrupt record is refused naming its path; an invalid board `creel.yaml` is refused naming `lym config creel`; a `seam.registryDir` returning an error naming `XDG_STATE_HOME` and `HOME` is refused naming that cause.
`rig.addHub` gives a row its second hub: a `hubforge.CopyHub` whose `.lym-shortname` it rewrites, since `hubforge` hubs share `TestShortname`, with that hub's geometry, delivery and clock and the rig's one `seam`.

`Command`'s `Long`, `internal/creelcli/doc.go`'s verb list, `docs/overview.md`'s `internal/creelcli/` line and the list of `lym creel` verbs in its **creel** module bullet name `hubs`.

**ImpactSummary:** One new leaf verb under `lym creel`; `cmd/lym/clitree_test.go` walks it with no list to edit, and no existing verb changes.
