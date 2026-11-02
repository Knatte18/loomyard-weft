Task: apply the fix-proposal lens to the findings fixed as proposed (F1 held-root row, F3, F4, F5, F6). I used no extra tool calls.

**Findings**

1. **LOW** — `docs/overview.md`, adopt bullet, the new line "With root, when the prime's head is ahead of the running lym's build, the hub's landing deploy runs next…".
   - The sentence is narrower and looser than the code on two points.
   - The gate in `internal/adoptcli/adopt.go:333` is `rootGoing`, which is `!held[root] && (rootNamed || (owners.Prime == "" && len(adoptable) > 0))`.
   - A plain beam adopt on a hub with no prime owner also records the prime, so it deploys too, with no root named.
   - Naming root alone is also not enough to deploy. `readDeployState` returns an empty state unless the landing config has a `deploy` command and its `deploy_branches` list the prime's branch.
   - The reviewer's "with root" wording was carried over unchanged, and a reader would take it as unconditional.
   - A fix would say the deploy runs when the call records the prime's owner, and only when the landing config deploys into the prime's branch.

2. **LOW** — `pattern/PATTERN-cli-cobra.md:36`, the `adoptcli` line now reading "…`battencli`, `battenshed`, `buildvcs` and `landingshed`".
   - The line says "takeover functions of". `buildvcs` supplies a build-stamp read and `battenshed` supplies deploy types and a timeout. Neither is a takeover function, so the sentence is not true of them.
   - The list was never complete either: `deps.go` imports `orchcli`, `hubgeom`, `shedrun`, `gateslot` and others the line omits. Adding two names keeps a perishable enumeration alive without making it complete.
   - The reviewer's proposal is carried out faithfully, but it is a weak fix. Rewording the line to "calls the functions of the modules it takes over, listed by `go list -deps`" or similar would avoid the staleness.

**Proposals that held up**
- **F1 held-root row:** with `!held[root]` removed from `rootGoing`, the row's `staleBuild` makes `requireNoDeploy` fail, so it fails on the defect. A passing run with an empty `ownerReq` also shows the stop did set `held[root]`.
- **F6:** `finish()` returns `rec.Entries()` and `Partial` on the refusal path, and the exact `DeepEqual` fails if the refusal drops the reconcile's entry. `w.report` is a per-world value, so `AppendRef` leaks nothing across parallel rows.
- **F5:** the extracted `deployFailure` rows assert exact strings. The log path and scratch directory in `runAdoptDeploy` stay unpinned, but the reviewer's `fix` scoped the change to the mapping only, so this is not a defect the fix introduced.
- **F2 second row:** `newWorld` sets `Prime: here`, so the renamed existing row does cover the already-owned case.

**Outside the lens:** F2 was fixed with an identity seam instead of the proposed executable-path seam. As a result, `runningLymBuild`, the production `os.Executable` + `OfFile` wiring, is now reached by no `adoptcli` test, though `buildvcs` tests cover `OfFile` itself.

**Tool calls:** none.
