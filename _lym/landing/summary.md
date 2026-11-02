# lym adopt root deploys when the installed lym predates the prime's head

`lym adopt root` now runs the hub's landing `deploy` command when the lym that is running was built from a commit the prime's head is ahead of.
Before, the adopt reconciled the prime onto a newer `main` and let runs go on under the old binary.
Runs that depended on new code then failed; the observed case was Darn-Write failing with `stencil: unfilled top-level marker(s): subagent_model`, because the installed lym never fills a marker the new stencil needs.
The adopt check stayed green, since it builds its own test binaries.

## Behaviour

- The deploy step runs after the holds are placed and before the owner write, so the beam steps and the adopt checks the call starts already run the new lym.
- It runs only when the call records the prime's owner, the `landing` config has a `deploy` command, `deploy_branches` lists the branch the prime is on, and the prime's head is ahead of the running lym's build.
  This is the same gate the Land row applies.
- A deploy that fails, times out or cannot start refuses the verb before the owner write.
  The holds stay, so no run goes on under the old lym, and the message names the log and the re-run.
- A ran deploy shows in the envelope as `deploy`: the build it replaced, the prime's head and the log path (`deploy.log` in root's adopt scratch directory).
- When the state cannot be read, as with a lym that carries no VCS stamp, the adopt goes on with a line in `warnings` that names the hand deploy.

## Design decisions

- "Ahead" means the running lym's stamped revision is a commit the prime holds, differs from the head, and is an ancestor of it.
  A build of a commit the prime lacks, such as unpushed local work, is not treated as behind, so a developer's own build is never replaced.
- A refusal was chosen over a warning for a failed deploy, since a warning would let the runs go on and fail the way they did.
  An unreadable state only warns, since nothing can judge it and refusing would block every adopt under an unstamped build.
- `battencli.RunDeploy` and `battenshed.DeployTimeout` are exported so adopt reuses the Land row's command runner and timeout, and `fabricengine.Fabric.CommitExists` is new for the existence check.
- The deploy takes no land lock, so it can run beside a landing's own deploy; the deploy command is expected to install by rename, as the hub's own does.
- `runAdoptDeploy` refuses to run in a test binary, where the hub's shell command must not run.

The change is documented in `docs/overview.md`, the `internal/adoptcli` package doc and `--help`, and `pattern/PATTERN-cli-cobra.md`.

## To check by hand

- Run `lym adopt root` on a machine whose installed lym is older than the prime's head, and confirm the deploy runs, `deploy` appears in the envelope and the runs start under the new binary.
  The real deploy was not run; the tests stub it.
- Check the deploy gate's case of a held root and of owned rows (`deploy-gate-held-root-and-owned-rows-missing`, `internal/adoptcli/adopt.go`): that the deploy runs only when the prime's owner is recorded by this call.
- Check `readDeployState` in `internal/adoptcli/deploy.go` (`read-deploy-state-untested`): that its tests cover the config, branch and ancestry paths over a real hub.
