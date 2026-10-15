---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: tools/forkaudit/plan-writer.jq:11
    summary: the plan-card write pattern drops the .md suffix card 6 names
    fix: in plan-writer.jq, end the Write file_path test with the card file suffix, so the pattern reads "(^|/)" + $plan + "[0-9][0-9]-.*\\.md$" and matches the card's _lyx/plan/NN-*.md shape
---

Review range: `git diff $(git merge-base main HEAD)..HEAD`.
`_lyx/shed/glyph-audit-tool/status.json` gives `product.parent` as `main`, and `_lyx/loom/rework/` does not exist, so no live rework round narrows the range.
The Publish failure section reads `none`.

Checked against the plan's six cards:

- Card 1: the command line, exit codes, required-command check, `--work` bound and marker, temp-file-then-`mv` report, discovery and header all match.
  The `PATTERN.md` line is byte-identical to the one the card gives.
  The run-record literals sit once in the variable block of `forkaudit.sh`, and no `.jq` file or Go test spells them.
- Card 2: the marker table heads `extract.jq`, and its comment and the script header state the add-a-row rule.
  `proj` comes from the run name passed in, and the card count comes from the `--arg` plan directory.
- Card 3: the missing-metric rule, `(n=<k>)` and `n/a` cells, pooled sum-over-sum ratios and quartiles are in `report.jq`, and the `--since` and `--runs` rows assert the hand-computed cells.
- Card 4: batch resolution uses `git log` and `git show` only, over the run's own refs through bracketed `--glob` patterns.
  The trace join takes the last line before the fork's start, and the disagreement, `trace:<value>` and unrecognized rules match.
- Card 5: the class set, the `--code` then `--records` refusal order and the missed-file list match, and the table asserts each class and each `existed` value.
- Card 6: the weft commit, the plan-card reference kinds, the writer choice with the lower-session-id tie-break and the not-counted lines match.
  Per-type mechanical checks: the diff holds no Rename or Delete group, and the Create and Edit groups' targets all exist.
- Gates, run in this review: `lyx gate test --tags integration` over the package passes, `go vet -tags integration` is clean, and `TestTestBudget_PackagesStayWithinBudget` passes.
  The budget row `tools/forkaudit: 5` equals the package's five top-level tests, and the final card sets that row.
  The kept audit directories are unchanged against `main`, as the plan's last verify line requires.
- Test economy: each test pins a behavior no other test covers.
  The report rows share one placement but assert different behaviors, and the trace, pack and plan tests each own their section.
  No per-helper test exists.

### [NIT] Plan-card write pattern ignores the card's .md suffix

**Location:** tools/forkaudit/plan-writer.jq:11
**Issue:** Card 6 defines a plan card write as a Write to `_lyx/plan/NN-*.md`, spelled from the variable block.
The filter matches `<plan dir>NN-` with no `.md` anchor, so a Write to a non-markdown file such as `_lyx/plan/01-notes.json` would count toward a session's plan card writes and could pick the wrong writer.
The fixtures write only `.md` cards, so no test sees it.
