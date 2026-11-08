```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, covering what the diff's changes force outside the lines they touch.

**Set per change**

| Change | Set found | Covered |
|---|---|---|
| `landDeps.Describe` split into `DraftDescription` and `RecordDescription` | `internal/beamcli/deps.go`, `land.go`, `land_test.go` fake, `deps_integration_test.go` | Yes. No remaining reference to `.Describe(` or `Describe:` in `beamcli`. |
| `landingshed.WriteBeamDescription` and `BeamDescribeDeps` renamed | Grep over all `.go` and `.md` files: no live reference left. `internal/landingshed/doc.go`, `internal/beamcli/doc.go` and `beamdescribe_test.go` are updated. | Yes. |
| `describe` signature gains `ctx` and `draft` | Only caller is `cli.go:240`, updated. | Yes. |
| Describe output now includes the `pr update` keys (`number`, `url`, `head`) and a new `drafted` key | Only the tests read them (`describe_test.go`). No other consumer: `pr.go` and `land.go` do not parse the describe envelope. | Yes. |
| Follow-up wording (describe now updates the pull request) | `pr.go` `pullRequestFollowUp`, `cli.go` `pr update` help, `dobbyengine/announce.go`, `dobbyengine/doc.go`, `announce_test.go`, `contracts/orchrole/role-beams.md`, `docs/overview.md:794`. All updated and consistent. | Yes. |
| `judgeDescribed` fix text (`land.go:250`) | Now names `summary.md`, no longer `lym board upsert`. The unchanged messages at `pr.go:61` and `land.go:243` say "run `lym beam describe`", which still holds. | Yes. |
| "Project decisions" and `discussionparser.AddedDecisionTexts` | The `deps.go` import is removed. No other caller of `AddedDecisionTexts` outside the package and its tests. No stale mention of lym owning `summary.md` in `docs/`, `contracts/` or `README.md`. | Yes. |
| Orch instructions, `contracts/orchrole/role-beams.md` | The text gives the end result, every member PR, and no decisions. This is the one instruction file for the beam landing. The other "change description" hits (`contracts/stencils/darn`, `landing`, `loom`, the `darn` and `sampler` recipes, `webster-spec`) are about task runs and are unrelated. | Yes. |
| `pawlengine/retiredmove.go:174` | Names `lym beam describe` generically, still true. | Yes. |
| PATTERN entries | The diff changes no `PATTERN.md` or `pattern/` file. The commit message carries no `Pattern-Departure:` trailer, and none is needed. No PATTERN entry names `beam describe`. | Yes. |

**Findings:** none. Nothing the diff changes leaves a caller, doc, stencil, fixture or rule document wrong.

**Tool calls**
1. `grep` over `.go`, `.md`, `.yaml` and templates for `beam describe`, `WriteBeamDescription`, `BeamDescribeDeps` and "Project decisions". My context did not hold the files outside the diff.
2. `grep` for `AddedDecisionTexts`, `DescriptionFindings` and "change description", plus `PATTERN.md` and `pattern/` for `beam describe` or `summary.md`. This checked the remaining callers and the PATTERN fit.
