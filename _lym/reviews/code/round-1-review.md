---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize_pr.go:195
    origin: lens:code-correctness
    summary: A beam's Finalize merges any pull request found in the run's scratch record as it stands, closed or merged ones included, though D8 limits that branch to a scratch record holding an open pull request
    fix: In `pullRequestLanding` in internal/landingshed/finalize_pr.go, take the `beam == BeamWithoutPullRequest && found` override only when GitHub reports the scratch record's pull request open. Open the API with `pr.API(origin.URL)` and read `api.Status(ctx, rec.Number)`, judging a failed call with `fz.githubFailure`. For a closed or merged scratch pull request skip the override and run `resolveWithParentRule` and `landingForDecision` as for any other record, so a closed one under an `on` or `off` parent is dropped by `dropRecordedPullRequest` and lands locally, and a merged one lands through the already-landed path. Add rows to `finalize_pr_test.go` for a closed scratch record under `off` (no Reopen call, local landing) and a closed one under `required` (resolver decides), beside the existing open-scratch row.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize_pr.go:293
    origin: lens:code-correctness
    summary: Finalize judges whether a merged pull request lacks beam commits with a plain commit log, while `lym beam ready` excludes parent merges and records-only commits, so a beam that ready passed is decided as having none recorded and a required or merge parent opens a new pull request
    fix: Use one rule for both callers. Move the filtering of `MergedLacks` in internal/beamcli/deps.go (find the merged head on `fabricengine.BeamLineCommits`, skip `RecordsOnly` commits and merges whose second parent the parent branch holds) into one function in internal/fabricengine/beamline.go, such as `BeamCommitsPast(l, parentBranch, mergedHead)`. Call it from `MergedLacks` and from the `CommitsMissing` seam that internal/loomcli/landingdeps.go wires for a pawl run, leaving the task recipes' `f.CommitLog(merged, task)` as it is. Add a row to `finalize_pr_test.go` or a `landingdeps_test.go` case where the beam is past the merged head only by a parent merge-in under a `required` parent, expecting the already-landed path and no `FindOpen` or `Create` call.
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/beamcli/deps.go:356
    origin: lens:code-tests
    summary: The upstream branch of ParentUnmerged, which fetches the parent's upstream under the remote-refs lock and judges its tip, has no test row that fails when it breaks
    fix: In internal/beamcli/deps_integration_test.go, beside `TestHubLandDeps_ReadyReads`, add a case with a hub whose parent has an upstream and whose push skip is unset (a local bare origin, never a live one). Advance the upstream past the local parent tip and expect `ParentUnmerged` true for a beam holding the local tip. Then merge the upstream tip into the beam and expect false. Keep the existing `SkipPush` cases as the local-tip rows.
  - id: F4
    severity: LOW
    class: design
    location: internal/loomcli/pawlwiring.go:168
    origin: lens:code-tests
    summary: The production closures of the retired move and of the beam landing record (`pawlRetiredMove`, `commitLandingDirectory`, `beamLandingRecordSeams`) are asserted only as set or unset, so a wrong session role, commit pathspec or record drop fails no row
    fix: In internal/loomcli add one test over a temporary beam pair that runs `beamLandingRecordSeams` drop for a landing record holding a pull request and expects the record rewritten with no pull request and the landing directory committed, and one that runs `pawlRetiredMove` over a pair holding a scratch pull-request record and a ready record, with the GitHub read faked, and expects the landing record carrying the pull request, the scratch and ready records gone and one commit. Reuse the hermetic git fixtures the package's other integration tests use.
  - id: F5
    severity: LOW
    class: consistency
    location: contracts/orchrole/orch-role.md:198
    origin: lens:code-affected-set
    summary: The Landing halts section still tells the beam orch that Parent-Align runs and that no pull request waits for an approval, which no longer holds for a pawl run
    fix: In contracts/orchrole/orch-role.md under `## Landing halts`, scope the Parent-Align sentences at lines 198 to 202 to task runs, and add one bullet for a beam's Finalize halted on a pull request GitHub refuses (a pending check, a dismissed review, a closed pull request), naming `lym beam pr status <slug>` and `lym loom resume`, never `lym batten cancel`. Keep the existing Landing a beam paragraph as it is.
  - id: F6
    severity: LOW
    class: design
    location: internal/beamcli/land.go:239
    origin: lens:code-correctness
    summary: The ready refusal for a beam with no recorded pull request names GitHub's rules as the demand even when the halt comes from an unparsable github.com origin under an on or off parent
    fix: In `judgeNoPullRequest` in internal/beamcli/land.go, build the demand clause from the facts. Name beam.yaml when the mode is `required` or `merge`, GitHub's rules when `rulesRequired` is true, and otherwise drop the demand clause and word the refusal as the beam's origin not being usable for a pull request, naming `landingshed.OriginMissing(side.Origin)`. Add a row to `land_test.go` for an `on` parent with an origin that `ParseOwnerRepo` refuses.
  - id: F7
    severity: LOW
    class: design
    location: internal/pawlengine/retiredmove.go:55
    origin: lens:code-correctness
    summary: A move repeated after the scratch record was removed posts a notice that leaves out that the pull request stays open
    fix: In `MoveOffRetired` in internal/pawlengine/retiredmove.go, when `carryPullRequest` returns no carried pull request because the scratch record is already gone, read the landing record with `ReadLanding(deps.LandingPath, deps.LandingLockPath, deps.Slug)` and, when it holds a pull request, pass that one to `retiredMoveNotice`. Add a row to `TestMoveOffRetired` for a repeat after a failed commit that expects the notice to name the pull request.
---

### [MEDIUM] Scratch-record override merges closed or merged pull requests

**Location:** internal/landingshed/finalize_pr.go:195
**Issue:** For a beam with no pull request in its landing record, `pullRequestLanding` forces `Open` and `MergeAtOnce` whenever the run's scratch record exists, without reading GitHub. D8 names only a scratch record that holds an open pull request. A closed one, such as one the operator closed to land without, takes the `reopen` branch, so Finalize reopens and merges it even under an `on` or `off` parent where the resolver would have closed it and landed locally.

### [MEDIUM] Finalize and ready disagree on what a merged pull request lacks

**Location:** internal/landingshed/finalize_pr.go:293
**Issue:** D3 passes a merged pull request at `lym beam ready` when the beam is past it only by merging the parent in, and says Finalize lands it. `recordedBeamLanding` instead asks `CommitsMissing(status.HeadSHA, head)`, which the beam wiring in internal/loomcli/landingdeps.go answers with `f.CommitLog(merged, task)`. That list holds the parent merge and the parent's own commits, so the pull request is read as lacking commits and decided as with none recorded. Under a required or `merge` parent that finds or opens a new pull request. I read `CommitLog` and the existing `finalize_pr_test.go` rows, which fake `commitsMissing`, and found nothing that applies `MergedLacks`' exclusions on the Finalize side.

### [MEDIUM] Upstream branch of ParentUnmerged is untested

**Location:** internal/beamcli/deps.go:356
**Issue:** D3 refusal 3 reads the parent's upstream tip after a fetch under the remote-refs lock when the parent has an upstream and the push skip is unset. `TestHubLandDeps_ReadyReads` runs with the push skip set and no upstream, and the `land_test.go` rows fake the seam, so breaking the fetch or the upstream-tip ancestor check fails no row.

### [LOW] Production glue of the retired move and the landing record is asserted only as wired

**Location:** internal/loomcli/pawlwiring.go:168
**Issue:** `wiring_beam_test.go` asserts `RetiredMove` is set for a beam and nil otherwise, and `landingdeps_test.go` asserts the beam record seams are set for pawl only. `TestMoveOffRetired` covers `MoveOffRetired` through fakes. No row runs the closures themselves, so a wrong `runproc` role, commit pathspec or record drop passes every test.

### [LOW] Landing halts section still describes a flow pawl no longer has

**Location:** contracts/orchrole/orch-role.md:198
**Issue:** The section says Parent-Align and Finalize fix their own failures and that no pull request waits for an approval. A pawl run has no Parent-Align, and a pawl Finalize can now halt on a pull request GitHub refuses, with a way forward naming `lym beam pr status` and `lym loom resume`.

### [LOW] Ready refusal names a demand that does not exist

**Location:** internal/beamcli/land.go:239
**Issue:** The halt branch picks "GitHub's rules for the parent branch require a pull request" unless the mode is `required` or `merge`. `ResolvePullRequest` also halts for an unparsable github.com origin under any mode, so an `on` or `off` parent with no rule gets a refusal naming a rule that is not there.

### [LOW] Repeated retired move drops the pull request from its notice

**Location:** internal/pawlengine/retiredmove.go:55
**Issue:** After `RemoveScratch` succeeds and a later step fails, the repeated move finds no scratch record, so `carried` is nil and `retiredMoveNotice` omits that the pull request stays open, though the landing record carries it.

## Rejected

- lens:code-tests finding 2 at MEDIUM, as stated: `wiring_beam_test.go:103-104` and `:133-134` do assert that `shedPaths.RetiredMove` is set for a beam and nil otherwise, so the claim that nothing reads `RetiredMove` is wrong. The remaining gap, the closures themselves, is kept as the LOW finding F4.
- lens:code-tests finding 3, unconfirmed: I read the 11 added lines in `internal/loomcli/landingdeps_test.go`. They assert only that the beam seams are set for pawl and nil for tapestry, so the gap is real but is folded into F4 rather than raised twice.
- Declared PATTERN changes: all four entries land in `PATTERN.md` with the declared text, including the two shortened ones, so no finding is raised. `pattern/PATTERN-heddle-instructions.md` carries no leftover `(new)` mark. The `ghguard_test.go` allowlist adds `internal/beamcli/` and the `prkit` fake, which matches the `PATTERN-github-auth` change. The commits carry no `Pattern-Departure:` trailer.
