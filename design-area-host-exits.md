# Stage 2 of the closed area: window, hibernate and deploy as supervisor verbs, and lym ide refused inside

`area-host-exits` · Task · feature, termwindow, hibernate, batten, ide, deploy, reed · draft

Decision record: [Sketch](decision-area-host-exits.md)

Depends on: [`area-apparmor-barrier`](design-area-apparmor-barrier.md)

Filed from the Sketch `sandbox-closed-area-build`.

Takes: the core part's supervisor verb protocol and attach client; termwindow's Launcher.OpenAttach, hibernatecli's request and its systemctl spawn, battenshed's LandDeps.RunDeploy and its wiring, idecli; the operator's stop of D14 step 3
Delivers: the supervisor's window verb (direct child of the hub root, title built from the directory name, Konsole running the host's lym reed attach with no shell after it) with lym reed window routed to it inside an area; the hibernate verb running systemctl hibernate alone; the deploy verb recording a pending deploy from Batten's Land row with a notice; lym area deploy cloning https://github.com/<repository>.git into the area state directory, fetching the area file's deploy_branch, checking the commit is on it and running the area file's deploy command with the host's toolchain and caches, refusing with the way forward when repository, deploy or deploy_branch is missing, and lym area init writing deploy_branch as main (Added after Discussion, with its acceptance criterion that a rewritten origin and a hub record naming another branch change neither); lym ide refusing inside an area
Exports: the window, hibernate and deploy verbs and their clients

The part changes these packages:

- internal/areaengine (about 300 changed lines)
- internal/areacli (about 200 changed lines)
- internal/termwindow (about 50 changed lines)
- internal/reedcli (about 40 changed lines)
- internal/hibernatecli (about 40 changed lines)
- internal/battenshed (about 60 changed lines)
- internal/battencli (about 40 changed lines)
- internal/idecli (about 20 changed lines)
