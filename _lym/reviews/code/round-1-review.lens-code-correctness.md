```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes with a concrete input, including failure cases.

**Per function, case run, result**

- `redundancyReportNote` closure (`internal/loomcli/redundancynote.go:22`):
  - A read error gives the "could not be read" line. Correct.
  - No file gives the "no gate call" line, which `wiring_test.go:945` pins. Correct.
  - A `fabricengine.Open` or `HeadSHA` failure reaches `matchesHead` as an error and renders "could not be matched against HEAD". Correct.
  - `HeadSHA` exists at `internal/fabricengine/warpforward.go:26`.
- `renderRedundancyReportNote` (`redundancynote.go:48`):
  - A report with `RedGate` and a non-empty `Note` hits the red case first, so the gate key is named and `matchesHead` is not called. Correct.
  - A report with a `Note` and no red tier renders "judged nothing". Correct.
  - A zero `Written` time formats as `0001-01-01T00:00:00Z`. Harmless.
  - A matching report with an empty `Tests` list and no errors renders only the header line. Acceptable.
  - A contained test renders its covering tests. Correct.
  - A non-contained test with an empty `Reason`, such as a unique one, renders only the verdict. Correct.
  - A report with `Errors` still matches and lists the error lines. Intended.
- `RedundancyReportMatches` (`internal/gatecli/redundancyreport.go:119`) for a worktree with no parent branch:
  - `parentMergeBase` returns `found=false`, so the function returns `(false, nil)`.
  - `renderRedundancyReportNote` then says "was written for a tree other than HEAD's", which is wrong in that case. See finding 1.
- `ReadRubric` and `redundancyReportNote` (`internal/shedadapters/rubric.go:58`, `:79`):
  - A nil `read` returns `none` without parsing. Correct.
  - A rubric without the marker returns `none`, and `read` is not called. The `TestReadRubric` row panics if it is called, so this is pinned.
  - A rubric with the marker repeated: `slices.Contains` is true, `read` is called once, and every occurrence gets the same text. Correct.
  - A blank answer from `read` renders `none`. Correct.
  - A malformed template with a non-nil `read` returns a wrapped parse error, and `Fill` would have failed anyway. Correct.
  - The banner is stripped before parsing, so the marker scan and the fill see the same text. Correct.
- `BouncerConfig.RedundancyReport` and `rubricNotes` (`bouncer.go:1657`): a nil value passes through as nil and renders `none`. The closure is called on each seed and judge rubric read, so a mid-run commit is picked up. Correct.
- `burlerReviewEntry` (`entries_burler.go:95`) and `bouncerEntry` (`entries_bouncer.go:296`): both forward `env.RedundancyReport`, so both call sites are covered. Correct.
- `wiring.go:751`: the closure is created at wire time but reads nothing until a rubric fill. Correct.
- `envkit.nilLegal` now lists `RedundancyReport`, so a nil value is accepted. Correct.

**Findings**

1. LOW, class `design`, `internal/loomcli/redundancynote.go:63-64`:
   - A worktree with no parent branch makes `RedundancyReportMatches` return `(false, nil)`, which `redundancynote.go:63-64` renders as "written for a tree other than HEAD's".
   - That misstates the cause. The gate-test step writes a "no parent branch" note in that case, so the real case is rare, but the match path itself gives no reason.
   - Evidence: `redundancyreport.go:123-126` and the doc at `redundancyreport.go:118`.

No other defects found.

**Tool calls**

1. `grep` for the `HeadSHA` and `parentMergeBase` definitions: to confirm both exist and what they return. My context had only their call sites.
