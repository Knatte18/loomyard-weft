# lym adopt root deploys when the running lym predates the prime's head

`lym adopt root` reconciles the prime against origin, which can move the prime's `main` past the commit the installed lym was built from.
Nothing deployed after that, so the runs the adopt released went on under the old binary and failed on code it did not know: two darn runs halted at Darn-Write on a stencil marker the old lym never filled.
The adopt check did not catch it, since it builds its own test binaries.

## What changes

When the owner write records the prime on this machine, `adopt` now runs a deploy step between the hold and the owner write (`internal/adoptcli/deploy.go`):

- `readDeployState` reads the hub's `landing` config.
  With an empty `deploy`, or `deploy_branches` not listing the branch the prime is on, nothing more is read and nothing runs, the same rule the Land row follows.
- Otherwise it reads the VCS stamp of the running lym (`buildvcs.OfFile` on `os.Executable`) and the prime's head.
  `installedBehind` reads the build as behind when its revision is a commit the prime holds, is not the head, and is an ancestor of the head.
  A build of a commit the prime lacks, such as unpushed local work, or one off the head's history, is not behind.
- A behind build runs the landing `deploy` command in the prime through `battencli.RunDeploy`, under the Land row's `battenshed.DeployTimeout`, with its output in `deploy.log` in root's adopt scratch directory.
  The envelope's new `deploy` field names the build it replaced, the prime's head and the log.
- A deploy that fails, times out or cannot start refuses the verb before the owner write, naming the log and the re-run.
  The holds stay, so no run goes on under the old lym, and the re-run judges the build again.
- A state that does not read, such as a lym with no VCS stamp, adds a line to `warnings` naming the hand deploy, and the adopt goes on.

Because the deploy runs before the beam steps and the adopt checks, the dobby, orch and check the same call starts run the new binary.

## Supporting changes

- `battencli.runDeploy` is exported as `RunDeploy`, and `battenshed.deployTimeout` as `DeployTimeout`, so the adopt runs the deploy exactly as the Land row does.
- `fabricengine.Fabric.CommitExists` is a read-only delegation to `gitrepo.Repo.SHAExists`, so the adopt can tell a revision the prime lacks from one that is merely not an ancestor without running git outside fabric.
- The `adoptcli` package doc gains a `# The deploy` section and names `deploy` in the envelope, and `lym adopt --help` lists the step; its long help was tightened to stay under the 2000-byte cap.

## Design decisions

- The deploy runs only when root comes to this machine, since the deploy is a prime-side action and the Land row deploys only landings into the prime's branch.
  A beam adopt on a machine that already owns the prime deploys nothing.
- The step runs the full fix the board entry asks for, a deploy before any run goes on, rather than its fallback of a warning alone.
- Bound: the deploy takes no land lock, so it can run beside a landing's own deploy.
  The hub's deploy installs lym by building it beside its target and renaming it over the target, so a process starting lym meanwhile never finds a half-written file; two deploys at once were not tested.

## Tests

- `TestAdopt` gains rows for a deploy before the owner write, a failed deploy that refuses with the holds standing and no owner write, beam steps or check, no deploy for a current build, no deploy without a command, no deploy for a beam adopt under a prime owned here, and an unreadable state that warns and goes on.
- `TestInstalledBehind` pins the ancestry judgement: ancestor, modified ancestor, head itself, a commit the repository lacks, a commit off the head's history, no stamp, and an ancestry read error.
- `TestAdoptFields` pins the `deploy` envelope field.
