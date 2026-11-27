# Fabric core: add, clone, adopt, junctions, weft wiring and the hub workspace move to the core; the vscode import is cut

`fabric-core-add-clone` · Task · enhancement, fabric, testkit, ide, batten, heddle, reconcile, shed · draft

Decision record: [Sketch](decision-fabric-core-add-clone.md)

Depends on: [`fabric-core-pull-replay`](design-fabric-core-pull-replay.md)

Filed from the Sketch `fabric-git-free-core`.

Takes: the pull and replay part's tree, with every earlier seam and the installer
Delivers: the rules of add, addadopt, addadoptonly, addanchor, addparent, clone, checkout, checkoutstate, forkrecords, gitexclude, warpexclude, junction, junctionnames, pairbranch, pairparts, resetsubpathcheck, scratchlink, shortnamebinding, shortnamecommit, unwire, weftwiring and hubworkspace in internal/fabriccore; Add's rollback as the core's inverse action list over executed actions and pre-creation facts, the engine keeping createdToken and createdBranchToken for destroy.go, with a tier1 rollback on a real hub; add's, addanchor's and forkrecords' shedrun uses through the run-path seam; hubworkspace's vscode import cut by the workspace builder seam, installed by cmd/lym/main.go beside InstallPairFolderNamer and replacing InstallWorkspaceContent, called on every exit that regenerates the workspace today, a rollback included, with nil writing no file; the IDE module's builder and its test in internal/ideengine; builder installs in the tests outside fabric that read the hub workspace file, internal/ideengine/spawn_hub_integration_test.go and cmd/lym/prerunseams_test.go among them; internal/beamcli/dobbyleftover_integration_test.go dropping t.Parallel; the cmd/lym test that runs the binary on a hub with every seam installed; the D12 moves of the clone, worktreeseam and add_runrecords tests that drive fabriccli, heddleengine, hubreconcile, boardkit or shedrun. Under the parent's added entry 'The restructure cuts fabric's test code, it does not only move it', this concern's tests, the largest of fabric's, shrink: a test is kept or moved only where it alone guards a rule, at the lowest tier that can check it, duplicates and the same rule tested at several tiers are deleted, not moved, and the change description lists every deleted test and, for each kept or moved test, the rule it alone guards
Exports: fabricengine's workspace builder seam and its setter

The part changes these packages:

- internal/fabricengine (about 4100 changed lines)
- internal/fabriccore (about 2800 changed lines)
- internal/fabriclayout (about 200 changed lines)
- internal/testkit/fabrickit (about 150 changed lines)
- internal/fabriccli (about 300 changed lines)
- internal/ideengine (about 400 changed lines)
- cmd/lym (about 150 changed lines)
- internal/beamcli (about 10 changed lines)
- internal/heddleengine (about 300 changed lines)
- internal/hubreconcile (about 150 changed lines)
- internal/shedrun (about 170 changed lines)
