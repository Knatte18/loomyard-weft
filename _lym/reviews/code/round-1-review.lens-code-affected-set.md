```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change the diff makes, I found what depends on it and checked whether the diff left any of it wrong.

**Per change, the set found**

| Change | Set found | Covered? |
|---|---|---|
| `pullrequest.API` gains `SetDraft` and `Comment`, and `Status` gains `Draft` | Implementers: `Client`, `fakePullRequestAPI` (`prreview_test.go`), `closeRecordingAPI` (`pairteardown/teardown_test.go`). Callers: `prreview.go` only. Docs: `pullrequest/doc.go`. | Covered. |
| `PATTERN-github-auth` (`PATTERN.md:37`) | The entry lists "open, reopen, merge or close". `SetDraft` and `Comment` are called only from PR-Review, so the "only PR-Review, Finalize, pair teardown" limit holds. | Covered. `PATTERN.md` is untouched and no declared PATTERN change exists. |
| `Paths.ReopenRequest` and `ReopenRequestFileName` | The one production builder `hubgeom.PawlPaths` (used by `pawlwiring.go`, `beamcli/deps.go` and `landingdeps.go`), the `requireSeams` check, and the test and fixture builders in `world_test.go`, `envkit.go` and `shedrecipe/fixture_test.go`. | Covered. |
| `PullRequestDeps.Reopen` | Filled in `loomcli/landingdeps.go`, `envkit.go` and the prreview fixtures. Both `intentionallyNil` lists (`battencli/wire_seams_test.go`, `loomcli/wiring_seams_test.go`) carry the entry. `sharedbootstrap.go` only fills session seams, so it needs nothing. | Covered. |
| `CommitSubjects` becomes `Commits` / `BeamCommit` | The `NewBeamDescribe` requirement list, `beamwiring.go`, the `envkit`, `conformance_test.go` and both seam-list tests. A grep finds no remaining `CommitSubjects` identifier. One stale comment (F1). | Covered, apart from F1. |
| Pawl recipe `PR-Review on_reroute: Beam-Await` | `pawl-recipe.yaml`, `loomrecipe/doc.go:43-46`, `loomcli/doc.go:40`, and the recipe and wiring tests. `loomrecipe/doc.go:16` still says PR-Review is always a Halt (F2). | Covered, apart from F2. |
| Finalize commit title `(#N)` | `finalize_pr.go`, `landingshed/doc.go:253`, `finalize_pr_test.go`. No other doc or stencil states the title rule. | Covered. |
| New verb `lym beam reopen` | `cli.go`, `beamcli/doc.go`, `docs/overview.md:459`, and `land_test.go`. The only other CLI-listing mention of a beam verb is `README.md:88`, which names `ready` only. | Covered. |
| `RemoveReady`, `ReopenRequest` and its write, read and remove functions | `pawlengine/doc.go` updated. The record goes through `internal/state`, so `PATTERN-tolerant-records` holds. The file sits under `.lym/beam/` beside the land request, so `PATTERN-durable-vs-ephemeral-state` holds. | Covered. |
| Beam-Describe `## Changes` list | `beamdescribe.go`, `landingshed/doc.go:275`, `loomcli/doc.go`. No stencil or spec in `contracts/` describes the old bullet-per-subject shape. | Covered. |

Nothing else in the set is wrong: no declared PATTERN change exists to land, and the commit messages carry no `Pattern-Departure:` trailer.

**Findings**

**F1 — LOW, consistency**
- Location: `internal/loomcli/beamwiring.go:24`
- Summary: the doc comment of `beamDescribeDeps` still says the empty-`parentBranch` refusal happens "when the commit subjects are read". The seam is now `Commits`, and it reads whole commits, not subjects.
- Evidence: `beamwiring.go:24` against `beamwiring.go:57-61`, where `beamCommits` returns the refusal.

**F2 — LOW, consistency**
- Location: `internal/loomrecipe/doc.go:16`
- Summary: the doc line "PR-Review's Stuck is always a Halt, so it carries no on_stuck, and Finalize declares no reroute" is now false for the pawl recipe. PR-Review's Stuck can be a Reroute to Beam-Await, as `pawl-recipe.yaml:74` and `doc.go:45` of the same file say. The diff edited the pawl paragraph further down this file but left this generic line.
- Evidence: `loomrecipe/doc.go:16` against `loomrecipe/doc.go:43-46`.

**Tool calls**
1. Bash (several greps): leftover `CommitSubjects` and bullet-per-subject wording, stale "no reroute edge" and "always a Halt" claims, the seam-list files, commit-title wording, `PATTERN.md` github-auth and reopen mentions, and beam verbs in docs. My context held the diff only, not the rest of the repo.
2. Bash (greps, one `sed`): `sharedbootstrap.go` around `RegisterSession`, `beamwiring.go:20-26`, and the remaining "always a Halt" claims. I needed to confirm there is no third seam list and to see the stale comment's wording.
