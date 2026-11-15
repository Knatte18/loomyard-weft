# lym adopt runs in the foreground over pushed state, with --take-hold, and the dobby tears down or reports leftover pairs

`adopt-foreground-leftovers` · Task · feature, batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle · draft

Decision record: [Sketch](decision-adopt-foreground-leftovers.md)

Depends on: [`bolt-holds-catch-up`](design-bolt-holds-catch-up.md), [`adopt-build-revision`](design-adopt-build-revision.md)

Filed from the Sketch `adopt-redesign`.

Takes: the holds, registry, catch-up and owner setter from bolt-holds-catch-up; the takeover entry type, its reader and the synchronous pushes from push-before-act-takeover; runfsm's states; adoptcli's fix loop, check, deploy, archives and here paths; the dobby's ended-run teardown and pairteardown
Delivers: lym adopt running hold, owner, pull, takeover, catch-up, deploy, workspace, check and green clear in the caller's process, re-runnable from tracked state, with the takeover entry writer and its rejected-push retry, and --take-hold denied to agents by a shell-guard row; the worker, adopt record and its readers, verdicts, pause step, the adopt and adopt-hold pause callers, --keep-local, --drop-local and --discard retired, with the upgrade's first read turning standing adopt entries into pause intents and one Bolt hold; the dobby tearing down landed and unkept cancelled pairs, listing kept pairs, writing kept-cancel records for pre-upgrade kept pairs, and tearing down idle orphans after the orphan grace; lym tender resume pushing no pair; the hub workspace file regenerated after each teardown
Exports: adopt's foreground step runner, the takeover entry writer and the orphan grace constant

The part changes these packages:

- internal/adoptcli (about 2650 changed lines)
- internal/adoptrecord (about 400 changed lines)
- internal/dobbyengine (about 350 changed lines)
- internal/fabricengine (about 150 changed lines)
- internal/beamcli (about 150 changed lines)
- internal/orchcli (about 80 changed lines)
- internal/pauserecord (about 80 changed lines)
- internal/battencli (about 60 changed lines)
- internal/shellguard (about 30 changed lines)
- internal/tendercli (about 30 changed lines)
- internal/pairteardown (about 30 changed lines)
- cmd/lym (about 10 changed lines)
- pattern (about 10 changed lines)
