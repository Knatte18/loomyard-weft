```yaml
lens: code-affected-set
outcome: reported
```

**Task:** lens code-affected-set. For each change, I looked for the callers, tests, docs, stencils and fixtures it forces outside its own lines.

## Per change

| Change | Set found | Covered? |
|---|---|---|
| `ReviewInput` seam → `ReviewInputs`, plus `generationRecord.ReviewInputs` | All Go uses are on the new name: `Env`, `BouncerConfig`, `entries_bouncer.go`, `envkit`, loomcli wiring and the wiring tests. `nilLegal` is updated. The old `review_input` record key still reads through `inputPaths()`. The spec line in `shed-recipe-spec.md` is updated. | covered |
| `Landing.PullRequest.ShedStatus` | `checkWiring`, both `intentionallyNil` tables, `envkit.LandingDeps`, the test fixtures and the real wiring in `landingdeps.go`. | covered |
| `Drift.ChangedFiles`, `Drift.CommitMessages`, `Drift.ReviewBase` | `driftVerifyEntry` nil checks, `driftTestEnv`, `envkit.driftDeps`, the battencli `intentionallyNil` table, `nilLegal["Drift.ReviewBase"]`, the loomcli `driftDeps` and the `wire` assignment. | covered |
| `Config.PRReviewSmallMax` | `ConfigSpec`, `template.yaml`, `templateDefaults` and `docs/overview.md`. | covered |
| `loom-fragment-loosened-check` | Registry entry, embed var, `markers_test`, and an include in each code-committing brief. These are the Webster and Weaver implementer and verify-fix briefs, the darn writer, the darn gate fixer, the sampler agent, the burler fix step, the landing fixer and the PR-Review brief. The card-level fork gate fixer (`gatefix-body-card`, `webster-fragment-card-fixer`) commits nothing, so it correctly has no include. | covered |
| Plain-include resolution in `ReadForEngine` | The PR-Review and sampler brief renders now use `ReadForEngine`. The burler fix step's tests use `FillWith` with the include. | covered |
| `on_reroute: Code-Bouncer` on PR-Review | The three recipes, the `tapestry-rows-before-motif.yaml` fixture, the `loomrecipe` doc, and the `TestRouting_ProgressAt` and `TestRecipe_PRReviewReroutesToCodeBouncer` tests. | covered, except F2 |
| `pr-findings` verb | `loomcli` registration, `resolveRunID` exemption, `doc.go`, `docs/overview.md`, the PR-Review brief, and a refusal test for each way forward. | covered, except F1 |
| Rubric `{{.review_input}}` marker | `RubricNotes`, `ReadRubric`, `rubricMarkerAllowlist` in `rubric_test.go`, and both `reviewRubricNotes` and `rubricNotes`. | covered |
| D3 Drift loosened rulings | The `driftengine` package `doc.go`, the judge stencil, the verdict render, and the records (`JudgedFile`, `LoosenedCheck`). | not covered in `docs/overview.md` (F1) |
| `PATTERN.md` and `pattern/` | The diff touches neither. The constraints section declares no changes. | nothing to land |
| Commit trailers | The review range's commit messages carry no `Pattern-Departure:` trailer. | nothing to judge |

## Findings

**F1 — LOW, class scope — `docs/overview.md:1085-1088`, `:1328-1334`, `:1361-1362`**
- The Drift section still says `Drift-Verify` and `Drift-Recheck` have a judge rule only on expected changes. It does not mention D3's loosened-check ruling, the new Stuck on an `uncaused` check, or the `Loosened-Check:` trailer.
- The PR-Review passage still says the row "asks once per pull request", while D4 re-arms the ask after a send-back.
- The diff added the `pr_review_small_max` and `pr-findings` text but left these three passages stale.
- Evidence: `docs/overview.md` grep shows no `Loosened` or `loosened` hit, and lines 1085-1088 and 1361-1362 are unchanged by the diff.

**F2 — NIT, class consistency — `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:620`**
- The comment still reads "because its Stuck is always a Halt", directly above the newly added `on_reroute: Code-Bouncer`.
- Every other copy of that comment was reworded.
- Evidence: the `always a halt` grep hit at line 620, and the diff adds `on_reroute` at line 624.

## Tool calls

1. `grep` for stale `ReviewInput` references, "always a Halt", `pr_review_ask_parent` and `pr-continue` docs, `stencilstore.Read` callers, and stencils that mention commits but lack the fragment. My context did not hold the repo-wide caller sets.
2. `grep` and `cat` for the card-gate fixer users and commit lines, the fragment's name references, and the `ReadForEngine` sites. I needed to know whether those briefs commit code.
3. `cat` of `pattern/PATTERN-run-records.md`, the friction API, `NotePath` users, the `docs/overview.md` Drift and loom lines, and a `git diff --stat` of `PATTERN.md` and `pattern/`. I needed these to judge the friction-note writer, the docs gap, and declared PATTERN changes.
