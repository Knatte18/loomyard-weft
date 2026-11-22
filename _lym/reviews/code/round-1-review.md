---
verdict: APPROVED
---

Range reviewed: `a042662792534f728cc1069f23d9830755b71af8..HEAD`, which holds commits d95d266ff and 4d8a01a2e.

The diff makes the change the record's `## Goal` asks for.
`hardenLiveArgs` gives a bare entry such as `internal/a` a `./` prefix and passes `./`, `../`, `.` and `..` entries through unchanged.
`TestHardenLiveArgs` pins both forms with literal expected values, in the plain call and in the `--copies` call.
`hardenLiveRunner` is the only caller, and it hands the args straight to `lym gate test`.
The gate itself builds `./<dir>` package arguments (`internal/gatecli/tierrun.go:803`), so the prefixed form is one it already resolves.
Every other reader of `hardener.packages` takes the raw config entries, not the arguments: `CheckCampaign`, `copiesRun`, `packagesOutsideStrayCover` and the `Started == 0` check.
None of them changes.

The `Root-Cause:` trailer on d95d266ff names the defect the diff removes: entries forwarded verbatim, so go read a bare directory as a standard-library import path.
The doc comment and the commit message agree with the code.
The change description's claims hold against the diff.
Its `two-way` merge-danger call is sound: the diff changes no record format and no config meaning, deletes no data and pushes nothing.

## Rejected

- lens:code-scope LOW, scope: an absolute entry gets the prefix, and the record licenses it only for a relative path. Rejected because an absolute entry never reaches `hardenLiveArgs`. `Config.Packages` is documented as worktree-relative directories (`internal/hardenshed/config.go:31`). `CheckCampaign` stats `filepath.Join(WorktreeRoot, pkg)`, which turns an absolute entry into a path under the root that does not exist, so Preflight refuses it (`internal/hardenshed/campaign.go:55-58`). The live gate reads the same checked config (`internal/hardenshed/campaign_rows.go:88`). A `filepath.IsAbs` guard would handle a state the code already rules out.

## Notes

- Focus question on edge inputs: an absolute entry and a `...` pattern are both refused by `CheckCampaign` before the live call. An empty entry becomes `./`, the root directory that `CheckCampaign` accepts for it. The record's "relative path that lacks it" covers every entry that can reach the function.
- `lym board get harden-live-bare-package-paths` failed with an invalid-json error. I read the task's board entry from the record's `## Goal`, a copy of the entry body, instead.
