# A machine registry and adopt holds in Bolt, and a catch-up from origin after every absence

`machine-registry-holds-catchup` · Task · feature, batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut · running · Code-Bouncer

Decision record: [Sketch](decision-machine-registry-holds-catchup.md)

Filed from the Sketch `adopt-build-resplit`.

Takes: transition-push-takeover's synchronous Bolt writes, takeover reader, hand-over, hand-over mark and owed-push facts, and its pair-push entry taking the set of names reading as this machine; runfsm's Held and OwnedElsewhere inputs, Judge and Aside, and pairteardown.ReadRun; pairteardown.EndSubjectProcesses and its local mode; boardengine's owner setter and OwnedElsewhere; machinename.File, FilePath and Names and lym machine name's owner sweep (internal/machinecli/set.go); dobbyengine's OwnedWorktrees, EnsureLive, loop, handover and PauseOwned seam, its OriginOwed passes; tendercli.PauseOwnedElsewhere; pauserecord's callers, CoveredError and SleepWatch; battencli's origin-owed marker; hibernate's wake hold of runs owned elsewhere; the fix-tries.json reader and writer in internal/adoptcli/holds.go (the unexported readTries and writeTries) and their callers in adoptcli's fix loop, green clear and ResetTries; fabricengine.AdoptScratchDir; dobbyengine's import allowlist (internal/dobbyengine/seam_enforcement_test.go)
Delivers: board.json's machines map and holds list written only by boardengine's setters with a synchronous push; a random machine id in machine.json kept by every writer, forgotten names and lym machine name --forget-former; registration at the first catch-up, dobby start, owner write and lym machine name with every D3 refusal and its way forward; every machine-name comparison through the registry, OwnedElsewhere, OwnedWorktrees, EnsureLive and ReadRun included, and the name set handed to fabricengine's pair push; the catch-up after a sleep, a dobby start or a hand pass, pushing owed commits, handing over runs owned elsewhere and removing their local pairs in local mode; the origin-owed marker retired; ReadRun filling Held, false for the adopt-fix hold's fix task, and the held refusal naming the hold's machine; bolt-hold slug entries under pauserecord.BoltHoldCaller, read by pauserecord's readers and worded by CoveredError; the owner caller and its writers retired into hand_over; the upgrade's first read turning standing adopt and adopt-hold entries into pause intents and one Bolt hold, run by the dobby start before its catch-up in internal/dobbyengine; the fix-tries.json reader and writer moved out of internal/adoptcli/holds.go into internal/fixtries (new) as fixtries.Read and fixtries.Write, importing only internal/state and taking the subject's directory from the caller, so neither internal/dobbyengine nor internal/adoptcli imports the other, adoptcli's callers switched to them and dobbyengine's import allowlist gaining internal/fixtries; the first read calling fixtries.Read on each subject's directory under the adopt scratch directory its caller hands in
Exports: boardengine's registry and hold types, their readers, and the one hold write the upgrade's first read calls, writing an adopt or adopt-fix hold; OwnedElsewhere taking the registry; the catch-up entry point the dobby start calls here and adopt calls later, run by the dobby start in internal/dobbyengine; pauserecord.BoltHoldCaller; fixtries.Read, called by the upgrade's first read and adoptcli's fix loop, and fixtries.Write, called by adoptcli's fix loop, green clear and ResetTries

The part changes these packages:

- internal/boardengine (about 450 changed lines)
- internal/dobbyengine (about 450 changed lines)
- internal/machinecli (about 300 changed lines)
- internal/pauserecord (about 160 changed lines)
- internal/battencli (about 140 changed lines)
- internal/orchcli (about 120 changed lines)
- internal/machinename (about 100 changed lines)
- internal/tenderengine (about 100 changed lines)
- internal/beamcli (about 100 changed lines)
- internal/pairteardown (about 60 changed lines)
- internal/tendercli (about 60 changed lines)
- internal/hibernateengine (about 60 changed lines)
- internal/runfsm (about 50 changed lines)
- internal/adoptcli (about 40 changed lines)
- internal/fixtries (about 40 changed lines)
- internal/loomcli (about 40 changed lines)
- internal/battenshed (about 30 changed lines)
- internal/hubgeom (about 20 changed lines)
- internal/landingshed (about 10 changed lines)
- internal/fabricengine (about 10 changed lines)
- internal/shedrecipe (about 10 changed lines)
- internal/shedrun (about 10 changed lines)
- internal/hibernatecli (about 10 changed lines)
- internal/fabriccli (about 10 changed lines)
- internal/fronttask (about 10 changed lines)
- internal/pawlengine (about 10 changed lines)
- pattern (about 10 changed lines)
