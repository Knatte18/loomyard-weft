# The dobby tears down or reports leftover pairs

`leftover-pairs-teardown` · Task · feature, dobby, workspaces, landing, fabric, batten, tender, shed · running · Drift-Verify

Decision record: [Sketch](decision-leftover-pairs-teardown.md)

Filed from the Sketch `adopt-build-resplit`.

Takes: runfsm's landed and cancelled states and runfsm.Derive's reading of a cancel intent with its finish; dobbyengine's ended-run teardown (strandedEnded in internal/dobbyengine/ended.go); pairteardown's teardown and local mode; the hand-over mark from transition-push-takeover and its drop at adopt's takeover write from adopt-foreground; the holds, registry and owner reads from machine-registry-holds-catchup; D1's synchronous push for the kept-cancel intent; lym tender resume's pair push; fabricengine.RegenerateHubWorkspace
Delivers: the dobby tearing down landed pairs and cancelled pairs without keep, by FSM state, with no pause, hold or other owner covering them, a cancelled run's batten record going in the same step; the dobby removing in local mode the pair of a run its hand-over mark names while another machine owns it; kept pairs listed with their age in lym beam status and in one notice per dobby start naming lym fabric remove <slug>; a finished keep cancel intent for each pre-upgrade kept-cancel record at the first dobby start; orphans noticed, their first-sight time in a machine-local file named in internal/lymdirs, and torn down idle after the orphan grace with their branches archived; lym tender resume spawning no pair push for a transition; the hub workspace file regenerated after each teardown
Exports: the orphan grace constant and the first-sight file's path

The part changes these packages:

- internal/dobbyengine (about 250 changed lines)
- internal/beamcli (about 80 changed lines)
- internal/pairteardown (about 60 changed lines)
- internal/fabricengine (about 50 changed lines)
- internal/battencli (about 40 changed lines)
- internal/tendercli (about 30 changed lines)
- internal/shedrun (about 20 changed lines)
- internal/lymdirs (about 10 changed lines)
