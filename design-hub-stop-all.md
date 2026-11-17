# lym beam stop --all takes the hub down on this machine, behind a stop button

`hub-stop-all` · Task · feature, hibernate, workspaces, orch, reed · running · Code-Burler-Review

Decision record: [Sketch](decision-hub-stop-all.md)

Filed from the Sketch `adopt-build-resplit`.

Takes: hibernate's pause-and-confirm inline in carry (internal/hibernateengine/watch.go) with awaitConfirmation; beamcli's stop command and its start --all; orchcli.BeamDown and its callers (beam stop, a beam's cancel, pawl's Beam-Down row); runproc.EndSubject and RoleOrch; lym reed down's pair end (internal/reedcli/up.go); tenderengine.Reap and Lift; selfexec.LymBinary and fabricengine's logged detached spawn as the pattern; reed's prime button group in both templates; D1's pushed-or-owed transitions
Delivers: hibernate's pause-and-confirm as one exported function hibernate's carrier and the stop both call, owning only an entry it added; lym beam stop --all re-starting itself detached with a logged spawn and running D9's sequence: hub pause and confirm, each task pair's processes and reed session through reed down's path, orchcli.BeamDown for the prime and each local beam, the beams' reed sessions, unowned run processes as Reap ends them, its own pause entry lifted through Lift, the prime's reed session last; orchcli.BeamDown ending the orch's registered Claude process through runproc as its last step for every caller; a stop entry with a confirm text in reed's prime button group of both templates; no tracked record, no per-run intent, a standing Bolt hold left as it is
Exports: hibernateengine's exported pause-and-confirm function; beamcli's stop --all; BeamDown's added end of the orch's Claude process. It touches no internal/dobbyengine, internal/machinecli or internal/termwindow code, which machine-start-launchers changes beside it

The part changes these packages:

- internal/hibernateengine (about 180 changed lines)
- internal/beamcli (about 120 changed lines)
- internal/orchcli (about 70 changed lines)
- internal/reedcli (about 40 changed lines)
- internal/reedengine (about 15 changed lines)
