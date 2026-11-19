```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F6 change set for agreement with its surroundings (fix-consistency lens).

**Finding 1 — LOW — `internal/gatecli/doc.go:565-575` (`# Stray files`) is stale against the change**
- The section says the listing is taken "before and after" a run. It now has to say the first listing is taken once per tier run, before its first attempt, and that a rerun after a shortage or crash is checked against it (`untrackedBeforeRun` in `run.go`, `runTierChecked`).
- It says nothing about the names in the line being quoted (`strconv.Quote` in `strayFilesLine`).
- "A `--copies` run is listed once around all its copies" is still true. The doc does not say that the failure report now lists the stray-file failure as its own failed run, labelled `runLabel(run)` (`copies.go` `addFailedRuns`).
- The review's F3 and F2 premises both cite this doc section, so a reader of it would not learn how the fixed behaviour works. The fix did not touch `doc.go`.

**Finding 2 — LOW — `strayFilesLine` text: the stray-file line's way forward reads "remove the files by their paths" while the paths are now Go-quoted**
- A name with a newline, quote or non-ASCII character prints as a Go escape (`"a\nb"`), not as the raw path. The "remove the files by their paths" wording does not say this.
- The text is consistent with F5's intent. Mention it only if the doc update in Finding 1 describes the quoting.

No other disagreements:
- `runTier` callers at `run.go:590` and `copies_test.go:108` still match the unchanged wrapper signature.
- Both tests that call `runTierChecked` match its new signature.
- The doc-comment in `checkStrayFiles` mentions `strayLine`, matching the new field.

Calls made:
1. `git grep` for stray-file mentions in `.md` and `.go` files, plus `git diff --stat`, to find sibling docs the change touches. My context had no search of other files.
2. `sed -n 555,590p` of `internal/gatecli/doc.go`, plus a grep of `copies` in it. My context did not hold `doc.go`, which the review names as the home of the `# Stray files` rule.
