---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md:Design decisions
    summary: the change description says newWiredPairFixture has two sites, but grep finds fifteen call sites in internal/loomcli
    fix: In _lym/landing/summary.md, replace the claim that newWiredPairFixture "has two sites, below the three-site threshold" with its real reason for staying on CopyHub, or drop the claim. The task names only beamHub and newBadReedUpFixture, so the omission is in scope; the stated count is not true.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md:beamHub bullet
    summary: the change description says beamHub is used by six tests and subtests, but grep finds five call sites
    fix: In _lym/landing/summary.md, correct the beamHub site count to match the call sites in internal/loomcli/projectrecord_integration_test.go, or drop the count.
---

The diff moves `beamHub` and `newBadReedUpFixture` onto `hubforge.Registration` templates, as the decision record's D7 asks.
The two fixtures are the only ones the entry's Goal names, and the diff touches nothing else.
Each registration's build runs the former steps in the former order.
Each site's seed still runs on its own copy after `lymcwd.Resolve`, and no assertion changed.

Checked against the acceptance criteria and the focus directive:

- **Build steps and order.** `beamHubRegistration.Build` adds the beam pair, `member-with-pair` and `forked-child`, then writes `board.json` and the shortname record, as `beamHub` did.
  `badReedUpRegistration.Build` corrupts the `mouse:` line, runs `SeedConfig` with the same map, then `AddPair`, as `newBadReedUpFixture` did.
  The `.` base shape has no pairs, so the seeded config commits before the pair forks from it.
- **Uncommitted board state.** `CopyRegistered` stages only the `.lym-warp` binding before it amends, so the uncommitted `board.json` and shortname record survive in each copy.
  `requireBoardAtUpstreamTip` compares commits only, so they do not trip it.
- **Baked values.** The seeded config and the pair hold no template-root path, and `relocateHub` fails a copy that still names the root.
- **Process-wide calls.** Neither build calls `os.Setenv`, `os.MkdirTemp` or `git config --global`.
  Neither keeps `tb.Context()` past the build, calls a package-global seam setter or reaches code a once-only registration changes.
  `templateBuildTB` also refuses `Cleanup`, `Setenv`, `TempDir`, `Chdir` and skips.
- **Seams.** No test that calls `beamHub` or `newBadReedUpFixture` installs a process-global seam.
  `TestRunOwnerVerdict` installs a Bolt replayer but uses `CopyHub` only, and the builds push nothing the replayer would run on.
- **Keys.** `loomcli-beam-hub` and `loomcli-bad-reed-up` are unique and cannot collide with the shape names.
- **Tests.** The diff adds no test, so there is no redundancy.
  Existing tests pin each behaviour: `TestMemberPairs_ReturnsEveryLocalPairUnderTheBeam`, `TestDecisionReceivers_ReachABeamsMemberPairs`, `TestDecisionParent_ABeamsRecordIsItsOwnOrchs`, `TestCommitRunRecord_ProjectRecordCommitsItsDirectory`, `TestLoomFailedReedUpRefuses` and `TestLoomResumeReconcilesTheHubConfigBeforeArming`.
- **Commit messages.** They match the diff, and no `Pattern-Departure:` trailer is needed.
- **Summary claims.** The four-site count for `newBadReedUpFixture` holds.
  The other two site counts do not: F1 and F2.

### [LOW] newWiredPairFixture site count is false

**Location:** `_lym/landing/summary.md`, "Design decisions".
**Issue:** The description says `newWiredPairFixture` "has two sites, below the three-site threshold".
Grep finds fifteen call sites across `smoke_test.go`, `smoke_drivingsurface_test.go`, `smoke_prebootstrap_integration_test.go`, `smoke_bootstrapwiring_test.go`, `smoke_gate_test.go`, `smoke_attachprobe_test.go`, `smoke_resumearming_test.go` and `smoke_starttail_test.go`.
The fixture's exclusion is in scope, since the Goal names only two fixtures, but the stated reason is not borne out by the code.

### [LOW] beamHub site count is false

**Location:** `_lym/landing/summary.md`, the `beamHub` bullet.
**Issue:** The description says `beamHub` is used by "six tests and subtests".
Grep finds five call sites, at `projectrecord_integration_test.go:72`, `:91`, `:123`, `:140` and `:156`.

## Rejected

- Two literals of `"loom-smoke-task"`, one in `newWiredPairFixture` and one as `badReedUpSlug`: the former is outside the task's scope, and both are fixture-local, so nothing goes stale between the code and prose.
- No loomcli test pins the registration keys' uniqueness: `hubforge`'s `slotFor` fails a duplicate at run time and its own tests cover it.
