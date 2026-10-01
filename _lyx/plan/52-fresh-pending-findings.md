# Card 52 — fresh-pending-findings

**Edit:**
- `internal/websterengine/runlevel.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/run.go`
- `internal/websterengine/runlevel_test.go`

**Uses:**
- `internal/websterengine/suspect.go`
- `internal/websterengine/archive.go`

**Intent:**
Fix PR #321 round-3 finding A's fourth item: `run --fresh` builds a fresh `State` before the pending-findings check, so it drops every pending finding while the suspect write is still in the tree.
It also only acts on a plan-fingerprint mismatch, so on an unchanged plan it cannot clear the pathless finding card 51 sends to it.

In `runlevel.go`'s `Run`, between the entry-time reclaim and the state `switch`:

- When `opts.Fresh` is set and the loaded state carries pending findings, `Run` runs `checkSuspectPaths` over every pending path with `runStartCommit(st)` as base, falling back to the worktree's HEAD when no batch recorded a start.
  A differing plan file does not refuse, since a fresh run adopts the on-disk plan as its new plan by design.
  Any other differing path refuses before anything is archived, wrapping `ErrPendingAuditFindings`:
  `--fresh would drop pending audit findings while their suspect paths still differ from the run's start commit <sha>: <paths>; way forward: reset the branch to <sha> with git, then re-run "lyx webster run --fresh"`.
  Unverifiable paths and pathless findings do not refuse either: the fresh run archives `state.json` and the reports dir, which is where they point, and starts from the reset branch.
- With pending findings and `opts.Fresh`, the archive and re-init branch runs whether or not the fingerprint matches.
  On an unchanged plan without pending findings, `--fresh` stays a no-op and the run resumes.
- The re-initialised run returns one warning per dropped finding, `--fresh dropped pending audit finding <id>: <detail>`, on `RunResult`'s warnings, so the drop is never silent.

The existing pending-findings refusal after the `switch` is unchanged, so a run without `--fresh` still refuses.

`internal/webstercli/run.go`: the `--fresh` flag's usage and `Long` say it also discards pending audit findings once their suspect paths match the run's start commit, and refuses while any differs.

`doc.go`'s run-entry paragraph states the `--fresh` rule.

Tests in `runlevel_test.go`, over the existing run fixture with one recorded batch whose `StartSHA` is the fixture's first commit:

- `TestRun_FreshRefusesWhileSuspectPathDiffers`: a pending finding naming a tracked file the test edits in the worktree, an unchanged plan, `Fresh: true` → `ErrPendingAuditFindings` naming the path and the start commit, `state.json` and the reports dir not archived, no Master spawned.
- `TestRun_FreshDropsFindingsOnceReset`: the same after `git reset --hard <start>` → the run spawns Master, the new `state.json` carries no pending finding, and the warnings name the dropped identity.
- `TestRun_FreshDropsPathlessFinding`: a pathless pending finding on an unchanged plan → the run re-initialises and proceeds, with the drop warning.
- `TestRun_FreshOnUnchangedPlanWithoutFindingsResumes`: no pending finding, an unchanged plan → the existing state is kept, its `RunGUID` unchanged.

**ImpactSummary:** run --fresh refuses while a pending finding's suspect path differs from the run's start commit, and otherwise discards pending findings with a warning per finding, even on an unchanged plan.
