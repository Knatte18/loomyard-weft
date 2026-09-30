# Card 7 — loomcli: tell landing the plan's verify command and scratch paths

**Edit:**
- `internal/loomcli#landingDeps`

**Create:**
- `internal/loomcli/landingdeps_verify_test.go`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `plan:internal/loomengine#LoomVerifyPendingPath`
- `plan:internal/loomengine#LoomVerifyOutputPath`
- `internal/planparser#ParsePlan`
- `internal/planparser#PlanDir`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`

**Intent:**
Fill the three new `landingshed.Deps` fields in `landingDeps`, so the gate the landing producers carry is live in loom:
- `VerifyCommand` is a closure that calls `planparser.ParsePlan(planparser.PlanDir(l.AnchorPath()))` each time it is invoked and returns the parsed plan's `Verify` — the already-chained single command line webster runs.
  It is a closure rather than a string because `landingDeps` runs at bootstrap, before the plan exists on a fresh run, so a value read there would be empty and the gate would silently skip.
  A read or parse error is returned wrapped (`loom: read plan verify command: %w`), never mapped to an empty command; only a parsed plan with no `## verify:` section yields `""`.
- `VerifyPendingPath` is `loomengine.LoomVerifyPendingPath(l)`.
- `VerifyOutputPath` is `loomengine.LoomVerifyOutputPath(l)`.
Comment the closure with the bootstrap-ordering reason above, matching the surrounding closures' comment style.
`landingDeps` itself still performs no I/O; only the closure reads, and only when called.

The reflection-based `TestLandingDeps_EveryFieldPopulated` drift guard already covers the new fields without an edit: it fails if any of the three is left zero.

`landingdeps_verify_test.go` (untagged; it only writes files under `t.TempDir()`) builds `Deps` through `landingDeps` with a `lyxcwd.Location` whose `HubPath` is a temp dir, then:
calls `VerifyCommand` before any plan exists and asserts a non-nil error;
writes a plan overview (a minimal `00-overview.md` with a one-entry Card Index and a matching card file, plus a `## verify:` section of two command lines) into `planparser.PlanDir(loc.AnchorPath())` after construction, calls the closure again, and asserts it returns exactly the two lines joined with ` && ` — proving the plan is read at call time;
rewrites the overview without a `## verify:` section and asserts `("", nil)`;
and asserts `VerifyPendingPath` and `VerifyOutputPath` equal the two `loomengine` accessors' results for the same location.

In `contracts/specs/loom-plan-spec.md`'s `## verify model` section, add a short paragraph after the one describing the chained command line:
landing is the verify line's second consumer — after a parent merge-in that changed the task tree, the `Publish` and `Finalize` landing producers run the same chained command in the task worktree and halt Stuck on failure before anything is pushed or landed, so the section must cover what a parent merge can break.
Use semantic line breaks.

**ImpactSummary:** `landingDeps` fills three more fields; its callers and signature are unchanged, and the gate becomes live in loom's two landing rows.

**Commit:** 7: loomcli: tell landing the plan's verify command and scratch paths
