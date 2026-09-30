---
verdict: APPROVED
rationale: "Range main..HEAD (product.parent=main) re-read at 916bede09 after the fixer's six commits; F1-F6 are all fixed in the diff, comments follow CONSTRAINTS.md's Comment Line-Break Convention, per-card checks hold, and go build, go vet (plain and smoke), go test over loomcli/fabricengine/shedrun/battencli and the integration MidMerge/ForeignMergeState/MergeVerbs run all pass."
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`, since `product.parent` is `main` in `_lyx/shed/loom-start-refuses-mid-merge/status.json`.
It covers the three card commits and six fix commits, one per finding.

The round-1 report returned APPROVED with six findings.
I re-read the whole diff at HEAD to check each one:

- **F1** fixed: the unconditional-evaluation sentence and the probe-kinds, test-pinning and kept-weft-probes paragraphs now sit on `readForeignProbes`.
  `foreignMergeStatePresent` keeps only its untouched first sentence and a pointer to the reader.
  Its answer and its four error strings are unchanged for all six callers.
- **F2** fixed: every new or changed Go comment in the diff is one sentence per line, and a line breaks further only at a comma or semicolon that begins an independent clause.
  The two pre-existing lines of `foreignMergeStatePresent`'s first sentence are untouched legacy lines, and CONSTRAINTS.md says untouched lines are never rewrapped.
- **F3** fixed: `assertNothingPutToWork` checks the starter, the sender and `loomengine.LoomDriverLog`.
  Every refusal row calls it: the spawn table, the parked-live row, the probe-error row and the real-probe row.
- **F4** fixed: `filepath.Dir(marker)` replaces `dirOf`.
- **F5** fixed: a `readForeignProbes` failure is wrapped as `fabricengine: mid-merge probe: read git merge state: %w`.
- **F6** fixed: a blank line now separates the test file's header from `package loomcli`.

Settled without a finding:

- The `default` arm of `refuseOverUnfinishedMerge` fails closed.
- Hoisting the marker stat keeps the resume, stale-marker removal and `driver_not_parked` paths unchanged.
- Nothing in `internal/loomcli` names either side of the pair.
- `SetsBothInjectedSeams` has no reference left outside `_lyx`.
- Both constructors set `midMerge` through `newLoomCLI`.

The report has no `## Focus departures` section, so there are no departures to ratify.
