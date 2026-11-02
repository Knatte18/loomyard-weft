---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/adoptcli/adopt_test.go (TestAdopt rows near the deploy rows), gate at internal/adoptcli/adopt.go:333
    summary: No TestAdopt row pins that a stop holding root back, or a prime already owned here, decides the deploy; dropping the held-root clause from rootGoing would pass every test.
    fix: In TestAdopt add a row with req Root true, w.deploy = staleBuild and the setup an existing row uses to hold root back by a stop, asserting requireNoDeploy and no owner write. Add a second row with req Root true, the prime already owned by this machine and w.deploy = staleBuild, asserting the deploy runs once before the owner write, since a re-run after a failed deploy must deploy.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/adoptcli/deploy.go:67
    summary: readDeployState has no test; the empty deploy short-circuit, the deploy_branches miss, the build-info read error and the wiring of CommitExists and IsAncestor into installedBehind are unpinned, since TestAdopt fakes DeployState.
    fix: Add TestReadDeployState in internal/adoptcli/deploy_test.go over a hubforge hub, as a table in the style of the package's other hub-backed tests. Rows are a landing config with no deploy (empty state, no executable read), a deploy whose deploy_branches omit the prime's branch (empty state), and a deploy for the prime's branch. Pass the executable path through a small parameter or seam so the rows can hand it a binary with a known VCS stamp, and give the build-info error row a path that is not a Go binary.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-cli-cobra.md:36
    summary: The adoptcli deviation line enumerates the packages adopt calls and omits battenshed and buildvcs, which the diff adds as imports.
    fix: In pattern/PATTERN-cli-cobra.md line 36 add battenshed and buildvcs to the list of packages whose functions adoptcli calls, keeping the line's wording otherwise.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:447-451
    summary: The adopt bullet describes the verb's steps in order and omits the new deploy step, which can refuse the verb.
    fix: In the adopt bullet of docs/overview.md, between the holds and the owner write, add one sentence that with root, when the prime's head is ahead of the running lym's build, the hub's landing deploy runs, and a failed deploy refuses with the holds standing.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/adoptcli/deploy.go:121-140
    summary: runAdoptDeploy is unreachable from any test because of its testing.Testing guard, so its scratch directory, log path and mapping of start error, timeout and exit code to messages are unpinned.
    fix: Keep the guard for the real shell run but move the result-to-error mapping into a small function over a battenshed.DeployResult and an error, and cover it in deploy_test.go with rows for a start error, a timeout, a non-zero exit code and a zero exit code.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/adoptcli/adopt_test.go, row AFailedDeployRefusesBeforeTheOwnerWriteWithTheHoldsStanding
    summary: The failed-deploy row does not assert the mutation record or Partial on the refusal return, which the refusal text and the focus rely on.
    fix: In the check of that row assert got.Partial equals whether the world's mutation record is non-empty and that got.Mutations carries the reconcile's entries, so a refusal that drops them fails.
---

### [MEDIUM] No row pins the deploy gate's held-root and already-owned cases

**Location:** `internal/adoptcli/adopt_test.go`, gate at `internal/adoptcli/adopt.go:333`
**Issue:** The deploy runs under `rootGoing`, whose held-root clause and `rootNamed` branch decide whether a deploy happens.
No `TestAdopt` row combines a held root with a stale build, and none pins the re-run on a prime this machine already owns.
Removing `!held[root]` from the gate leaves every test green while a deploy runs for a root that is not going.

### [MEDIUM] readDeployState is untested

**Location:** `internal/adoptcli/deploy.go:67`
**Issue:** The state read is what makes the deploy happen on a real hub: the empty-command and branch-list short-circuits, the build-info read and the ancestry wiring.
`TestInstalledBehind` covers only the pure judgement, and every `TestAdopt` row fakes `DeployState`.
A break in the short-circuit order, such as reading the build before the config says nothing deploys, passes all tests.

### [LOW] PATTERN-cli-cobra's adoptcli line no longer lists what adopt calls

**Location:** `pattern/PATTERN-cli-cobra.md:36`
**Issue:** The line enumerates the packages `adoptcli` calls; the diff adds imports of `battenshed` and `buildvcs`, which it omits.

### [LOW] The overview's adopt bullet omits the deploy step

**Location:** `docs/overview.md:447-451`
**Issue:** The bullet walks the verb's steps and now lacks one that can refuse the verb.

### [LOW] runAdoptDeploy cannot be tested

**Location:** `internal/adoptcli/deploy.go:121-140`
**Issue:** The `testing.Testing()` guard is the repository's usual spawn guard, but it also takes the error mapping and log path out of reach of any test.

### [LOW] The failed-deploy row leaves the refusal's mutation record unasserted

**Location:** `internal/adoptcli/adopt_test.go`, failed-deploy row
**Issue:** The refusal text promises the holds stand, and the row checks them, but `finish()`'s mutation record and `Partial` on that return are not asserted.

## Rejected

- Lens code-correctness finding 1, that `os.Executable` returns a `(deleted)` path after the deploy renames the binary, so the later adopt check, dobby and orch spawns fail: Go's `os.Executable` on Linux trims the ` (deleted)` suffix (`$GOROOT/src/os/executable_procfs.go`), so the spawns resolve the replaced file at the same path, which is the new lym.
- Lens code-correctness finding 2, that a refused deploy returns before `reconcileWarnings` and `reportRuns`: the other post-hold refusals in `adopt` return at the same point, so the refusal envelope is as complete as theirs and the change is no departure.
- Lens code-tests NIT, `CommitExists` has no test of its own: a one-line delegation that a `readDeployState` test (F2) exercises through the public surface, so a per-helper test would duplicate it.
- Handler check, an installed build whose commit the prime lacks reads as not behind and adds no warning: the incident's build, `87553450ec14`, was a commit of the prime's own history, so it is an ancestor and deploys; a commit the prime lacks is unpushed local work that a deploy would overwrite, which the doc states as intended.

## Focus verdicts

- `rootGoing` gate: true when root is named and not held back, or when the board records no prime owner and a beam is adoptable; a stop holding root gives false, a prime already owned here still deploys under `adopt root`, and the gate stands before every owner write path. Correct, with the missing rows in F1.
- `installedBehind` and `readDeployState`: `vcs.revision` and `HeadSHA` are both the full 40-character form, so the equality is sound; a modified build of the head is not redeployed and a modified build of an ancestor is, which is a stated policy; the not-behind reads leave no stale install undeployed in the incident's case.
- Failure path after `deployStale` errors: `finish()` carries the mutation record and `Partial`, the holds stand, no owner is written, and `adoptFields` is built before the error check, so `deploy` and its log reach the caller. Correct.
- `testing.Testing()`, `subjectDir` and the prime's worktree as the deploy's directory match the sibling spawn guards and the Land row's `RunDeploy` call; the new `battencli` and `battenshed` imports follow the existing cli-to-cli imports, and the only rule document they touch is F3.
- Test coverage: the uncovered behaviours are named in F1, F2, F5 and F6; the new `TestAdopt` and `TestAdoptFields` rows duplicate no existing coverage.
- Help and doc: the step order (hold, deploy, owner write, beam steps, checks) and the `deploy` field name agree across `cli.go`, `doc.go` and the code; the help's byte cap is enforced by `cmd/lym/clitree_test.go`, a mechanical gate.
