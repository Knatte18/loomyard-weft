```yaml
lens: code-pattern-fit
outcome: reported
```

Task: the code-pattern-fit lens, checking the diff against the repo's PATTERN.md entries and CLAUDE.md rules.

**Rules checked**

- `PATTERN-sole-parsers`: held. The report is validated only in `ParseReport` (`report.go` `validateReportFields`). Card-id tokens are classified only in `planparser` (`renumber.go`).
- `PATTERN-tolerant-records`: held. `Digest.Renames` is `json:"renames,omitempty"`, so an older state without it reads as no renames. `ReportRename` has json tags. The `deviations` key stays accepted and ignored.
- `PATTERN-refusal-way-forward`: held. The `ParseReport` error for a bad rename entry is wrapped at `recordbatch.go:476` with the recover-batch way forward. A test row covers each bad entry (`report_test.go`).
- `PATTERN-stencil-ownership`: held. The implementer and master stencils state the `renames` field. The stencils are read at call time and not embedded.
- `PATTERN-test-speed`: held. Every new or changed test file carries a `tier` tag: `tier0` for `drift_test.go`, `renumber_test.go`, `recoverbatch_test.go` and `report_test.go`, and `tier1` for `recordbatch_git_test.go`. The tags are not "the lowest that fits" in every case, but I found no mismatch that needs git or quarry.
- `PATTERN-leaf-packages`, `PATTERN-gitrepo-client-boundary`, `PATTERN-fabric-git`, `PATTERN-shed-*` and `PATTERN-agent-name`: not touched by the diff.
- CLAUDE.md "Docs land in the same commit": held. `websterengine/doc.go`, `planglyph/doc.go`, `loomcli/doc.go`, the verbs' `--help` and the `loom-plan-spec.md` numbering section all change. No new module and no new cross-cutting invariant, so `docs/overview.md` and `PATTERN.md` need no edit.
- CLAUDE.md "Markdown: semantic line breaks": held in the stencils and the spec. Each sentence is on its own line.
- CLAUDE.md "Notes go in git": held. The diff writes no memory files.
- "Declared PATTERN changes landed": no declared changes appear in the directive, so there is nothing to check.
- "Commit messages and departures": held. Neither commit carries a `Pattern-Departure:` trailer, and I found no difference from the answer key that needs one.
- Comment conventions (`scribe:golang-comments`): held, with one cosmetic caveat. New comments sit above the code, not at end of line. `drift.go` and `renumber.go` file headers were rewritten and still describe the file. The `Digest` struct's existing end-of-line comments remain, but `Renames` gets an above-line comment.
- Stencil wording: held. The implementer stencil adds the plan-names-are-the-contract rule and the `renames` field with its `reason`. The master stencil lists `renames` as informational. I saw no rule dropped or weakened.

**Findings**

1. **LOW, design.** `ParseReport` also requires a non-empty `reason` (`internal/websterengine/report.go:~100`, `validateReportFields`). The decision record says to refuse an empty side or old equal to new. The `reason` requirement is an extra the record does not license, and it can refuse a fork report that is otherwise valid. The stencil does ask for a reason, so this is a deliberate tightening. Either drop the `reason` check, or record it in the decision as a scope departure. Evidence: `report.go` diff lines 775–778 and `decision-record.md` "Reported renames travel in the report and the digest".

No other rule is broken.

**Tool calls**
1. Bash: the `//go:build` tags of the five changed test files, the `ParseReport` callers, the `recover-batch` strings in `recordbatch.go`, and the diff of `report_test.go` and `recoverbatch_test.go`. My context had only the non-test diff.
2. Bash: `recordbatch.go` lines 470–490 and `recoverbatch.go` lines 155–170, to confirm where the `ParseReport` error gets its way-forward wording. The diff did not show those lines.
