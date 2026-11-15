# A machine registry and holds in Bolt, the owner pause caller retired, and a catch-up from origin after an absence

`bolt-holds-catch-up` · Task · feature, batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle · draft

Decision record: [Sketch](decision-bolt-holds-catch-up.md)

Depends on: [`push-before-act-takeover`](design-push-before-act-takeover.md), [`orch-resume-own-session`](design-orch-resume-own-session.md), [`adopt-build-revision`](design-adopt-build-revision.md)

Filed from the Sketch `adopt-redesign`.

Takes: runfsm's Act rule, the synchronous Bolt writes, the takeover reader and hand-over and the owed-push facts from push-before-act-takeover; the subject-process end from orch-resume-own-session; boardengine's owner setter and OwnedElsewhere, machinename's former names, pauserecord's callers and SleepWatch, the dobby's loop and handover, battencli's origin-owed marker, pairteardown's local mode
Delivers: board.json's machines registry and holds list written only by boardengine's setter with a synchronous push; every machine-name comparison through the registry, OwnedElsewhere's callers in every package updated; lym machine name --forget-former; hold slug entries under the new pauserecord caller hold, read by every machine; the owner caller and its writers retired into hand_over with the subject-process end; the origin-owed marker retired; the catch-up from origin after a sleep, a dobby start or a hand pass, pushing owed commits, handing over runs owned elsewhere and removing their local pairs in local mode, the dobby's handover calling the subject-process end
Exports: boardengine's registry and hold types, setters and readers, OwnedElsewhere taking the registry, and the catch-up entry point adopt calls as its step 4

The part changes these packages:

- internal/boardengine (about 600 changed lines)
- internal/dobbyengine (about 450 changed lines)
- internal/pauserecord (about 120 changed lines)
- internal/machinecli (about 150 changed lines)
- internal/orchcli (about 120 changed lines)
- internal/tenderengine (about 100 changed lines)
- internal/battencli (about 140 changed lines)
- internal/beamcli (about 100 changed lines)
- internal/tendercli (about 60 changed lines)
- internal/hibernateengine (about 60 changed lines)
- internal/adoptcli (about 40 changed lines)
- internal/machinename (about 40 changed lines)
- internal/loomcli (about 40 changed lines)
- internal/battenshed (about 30 changed lines)
- internal/landingshed (about 10 changed lines)
- internal/fabricengine (about 10 changed lines)
- internal/shedrecipe (about 10 changed lines)
- internal/shedrun (about 10 changed lines)
- internal/hibernatecli (about 10 changed lines)
- internal/fabriccli (about 10 changed lines)
- internal/fronttask (about 10 changed lines)
- internal/pairteardown (about 10 changed lines)
- internal/pawlengine (about 10 changed lines)
- pattern (about 10 changed lines)
