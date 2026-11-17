# lym adopt runs in the foreground over pushed state: hold, owner, pull, takeover, check, with --take-hold

`adopt-foreground` · Task · feature, board, dobby, fabric, workspaces, orch, tender, batten, process · landing · Finalize

Decision record: [Sketch](decision-adopt-foreground.md)

Filed from the Sketch `adopt-build-resplit`.

Takes: machine-registry-holds-catchup's hold types and readers, the hold write its upgrade first read calls, its registry types, readers and registration write with D3's refusals, its catch-up entry point, pauserecord.BoltHoldCaller, and fixtries.Read and fixtries.Write in internal/fixtries, which its upgrade first read and adoptcli's fix loop call; boardengine.SetOwners's pull-first commit and its re-read after a rejected push (internal/boardengine/ownerset.go); transition-push-takeover's takeover entry type, reader and synchronous pushes, and the hand-over mark's path and reader; runfsm's states and the refusals that name lym adopt <subject>; adoptcli's fix loop (fixloop.go, fronttask.File), check, deploy, archives and --here paths; the adopt record's readers in the dobby's handover, the orch watcher and lym beam status; orchcli.BeamUp; internal/shellguard's table
Delivers: lym adopt running hold and owner write, pull, takeover, catch-up, deploy, workspace, check and green clear in the caller's process, re-runnable from tracked state, with the same-set and other-set rules of this machine's own hold; boardengine's hold setters beside their callers here: rewrite a hold to name another machine keeping its kind, fix task, try counts and owners before the adopt, replace it with an adopt-fix hold, update try counts and clear it; one pull-first Bolt write in boardengine recording the adopt hold, or under --take-hold the rewritten hold, together with the subjects' owner and this machine's registry entry in one commit, which on a rejected push re-reads Bolt and refuses on an owner of these subjects or a hold another machine wrote since; the takeover entry's production writer and its rejected-push retry, step 3's takeover write dropping this machine's hand-over marks of each moved slug through a drop function added beside the mark reader; lym adopt --take-hold denied to agents by a shell-guard row whose steer and notice never spell the flag; lym adopt check reading the adopt-fix hold's try counts; --here keeping no_subject and owned_here; the worker, the adopt record and its readers, the held-back project verdicts, the adopt pause step, the adopt and adopt-hold caller writers and --keep-local, --drop-local and --discard retired; the dobby's handover and the orch watcher reading a hold; lym beam status showing the Bolt hold; adoptcli's and adoptrecord's package docs rewritten or the package removed, fixtries.Read in internal/fixtries staying for the upgrade's first read
Exports: adopt's foreground step runner, the takeover entry's production writer, and boardengine's hold setters and one-commit adopt write, each with its production caller in adopt

The part changes these packages:

- internal/adoptcli (about 2650 changed lines)
- internal/adoptrecord (about 400 changed lines)
- internal/boardengine (about 250 changed lines)
- internal/dobbyengine (about 150 changed lines)
- internal/fabricengine (about 120 changed lines)
- internal/beamcli (about 80 changed lines)
- internal/orchcli (about 80 changed lines)
- internal/pauserecord (about 80 changed lines)
- internal/battencli (about 60 changed lines)
- internal/shellguard (about 30 changed lines)
- cmd/lym (about 10 changed lines)
- pattern (about 10 changed lines)
