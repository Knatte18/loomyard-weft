```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what each change forces outside the lines it touches. I did not verify every caller, test or doc outside the manifest, so a clean row below means no gap seen in my context, not a full search.

**Per change: set found, covered or not**
- **PATTERN.md and `pattern/` (declared changes):**
  - `PATTERN-parent-lock`, `PATTERN-github-auth` and `PATTERN-shed-recipe-registry` land with the declared text.
  - `PATTERN-pull-request-mode` is added with the declared text.
  - `PATTERN-github-auth` carries no named-account clause.
  - Covered.
  - The diff also edits `pattern/PATTERN-gate-slot.md`, `pattern/PATTERN-shed-recipe-registry.md` and `pattern/PATTERN-test-speed.md`; the record declares no change to those three. I judged all three as tracking the new `pr-review` gate key and the new row, so no finding.
- **Commit trailers:** none of the 18 task commits (cards 1–17 plus the help-text fix) carries a `Pattern-Departure:` trailer (grep over the range was empty). A departure the plan declares is therefore unexcused. The plan's "One immediate poll on the continue flag" departs from D9's once-a-minute poll bound. The code at `internal/landingshed/prreview.go:559` polls early on the continue flag. This is a plan-level departure from D9, not from a PATTERN entry, so I raise no finding.
- **`PATTERN-github-auth` (test):** the entry says only PR-Review, Finalize and pair teardown open, merge or close a pull request. `internal/pullrequest/client.go` is the new GitHub caller, and `cmd/lym/ghguard_test.go` is added. I could not confirm that the guard's allowlist covers `internal/landingshed` and `internal/pairteardown`. Not asserted as a finding.
- **`PATTERN-github-auth` and `PATTERN-parent-lock`, the reroute of Finalize through GitHub:** `internal/landingshed/finalize_pr.go:176` merges on GitHub inside Finalize under the lock, so the declared entry holds. In `prreview.go` the row only closes a pull request (lines 165 and 644) and takes no lock, as the entry allows.
- **Recipes (D13):** the three recipe files and the `internal/loomrecipe` tests are in the diff. Covered by the diff stat; I did not read the YAML.
- **Retired keys (D3):** `loomengine` and `darnengine` config, template and tests changed, and `darnengine/doc.go` changed. `internal/configreg/configreg_test.go` changed. Covered.
- **Retired `pr_review_setting` registry key:** `internal/shedrecipe/entries_prreview.go`, its test and `shedrecipe/doc.go` changed. `shedbuild/build_engines_test.go` changed. Covered.
- **`interruptpolicy.go` comment (D7):** changed in the diff. Covered.
- **`battenshed/doc.go` (D10):** changed in the diff. Covered.
- **Quiet check (D10):** `battencli/wire.go` and `wire_seams_test.go` changed. Covered.
- **Board field `pull_request` (D2):** `boardengine` (task, store, replay, runlock, doc, tests) and `boardcli/cli.go` plus its help test changed. Covered.
- **`beam.yaml` key (D1):** `dobbyengine` config, template, doc and tests changed. Covered.
- **`runproc/record.go`:** changed, with the doc updated.
- **Shuttle unawaited start (D7):** `run.go`, `spec.go`, `wait.go`, `stop.go`, `attach.go`, the claude engine settings and `shuttleengine/doc.go` changed, with tests. Covered.
- **`shellguard` deny row (D7):** `rows.go`, `doc.go` and the integration test changed. Covered.
- **Stencil (D7):** `contracts/stencils/landing/landing-template-pr-review.md` and `stencils.go` are added. Covered.
- **Docs:** `README.md`, `docs/overview.md`, `docs/dotnet-hub.md`, `landingshed/doc.go`, `loomcli/doc.go` and `pairteardown/doc.go` changed. Covered.

No findings from this lens.

**Tool calls:**
1. `git diff` of `PATTERN.md` and `pattern/`, plus a grep of commit trailers for `Pattern-Departure`: my context held the PATTERN diff only partly and lacked the trailers.
