```yaml
lens: code-scope
outcome: reported
```

Task: map the diff to the decision record's Goal points (code-scope lens).

**Changes, and the decision each traces to**
- **C1:** commits b77b64d6 and 20ad9235. They trim 130 `doc.go` files: restatements, step-by-step narration, repetition, history, consumer lists and examples are cut, and the overview, contracts, invariants and why stay. This traces to G1 (cut superfluous text in every `doc.go`), G2 (what counts as superfluous) and G3 (what stays).
- **C2:** commits ebd33289 and ca09c04a. They re-break the trimmed comment lines at sentence and clause boundaries and fix the last comment-lint findings. This is forced by C1, since the repository's comment lint (`lym loom lint-comments`) must pass on the lines C1 changed. It is not unrequested scope.
- **Other changes:** none. Every changed file is a `doc.go`, and every changed line is a comment or blank, as the handler's mechanical checks found (empty output from the not-doc.go and non-comment-line filters). No commit carries a `Pattern-Departure:` trailer, and none needs one.

**Decisions, and the changes that build each one**
- **G1, cut superfluous text in every `doc.go`, not a chosen set:** built by C1. 130 of 162 `doc.go` files changed. My one tool call listed the files the diff leaves untouched: the longest is 17 lines (`internal/partsparser`, `internal/battenrecipe`), and the rest are 16 lines or fewer. No large file was skipped.
- **G2, what counts as superfluous:** built by C1.
- **G3, what stays:** a short overview, contracts and invariants, and the why. Built by C1. The cited headings checked in the manifest still exist: fabricengine's mutation record section, the Windows settling text, reedengine's `display-message` lines, and orchengine's `Continuing after a reboot`.
- **G4, how the work is done:** Sonnet subagents at medium effort, batches sized by file length, the largest files in their own batch, and the writer checking each subagent's diff. This is process, not something the diff can show. The only evidence is the writer's claim in `_lym/landing/summary.md` under "Design decisions", which matches it. No finding.
- **G5, no code changes and no comments in other files:** honoured. The diff touches only `doc.go` comment lines, which is the evidence C3 rests on.
- **`Added after Discussion` entries:** the record has none.

**Findings**

None for this lens.

One thing outside my lens, for the handler's residual-text review: `internal/batcher/doc.go` went from 135 to 128 lines. It still narrates the estimator's PeakContext formula term by term (`internal/batcher/doc.go`, the estimator paragraph). That bears on whether G1 was applied fully to that file, not on scope.

**Tool calls**
1. Listed the `doc.go` files the diff does not touch, with their line counts. My context held only the changed files' sizes, and G1 says "every doc.go", so the skipped files needed checking.
