# Drift: keep capture metadata out of the check and end unfixable gates

A capture's provenance, such as `_source.txt` with its commit and capture time, no longer fails the drift gate, re-prompts the writer or reaches the parent twice.
Drift now compares only what the code's results are, with no `volatile_files` or `volatile_keys` entry needed.

Closes #81

## What changes

- A capture command may use a `{meta_dir}` placeholder.
  It renders to the `.meta` sibling of the capture directory, which drift keeps with the capture and never lists, compares or shows to an agent.
  `drift init`, the template and `internal/driftengine/doc.go` say provenance goes there.
  The baseline directory holds the compared files only.
- A baseline backup moves its metadata directory with it, and a leftover metadata directory beside an empty baseline is removed.
  When the metadata move fails, the baseline moves back so the pair is never half-moved.
  When that move back fails too, both errors are returned and the pair stays split.
- The expect agent tells metadata from results.
  The stencil (`drift-template-expect.md`) and the expectation file take a `metadata` block with `files` globs and `keys`, checked against the baseline's listings.
  `diff: command` refuses `metadata.keys`, since that compare keys no line.
- The marks are recorded in the scope record at the seal and joined with the sealed expectation and its amendments.
  The check drops their cells after the diff, so no later listing, expectation, check or gate shows them or fails on them.
  A ruling on a case the marks name counts as `matches`.
- A drift failure whose unexpected cells no code change can move ends the gate at once as a Terminal failure, with no re-prompt.
  The findings go to the parent once, with `lym drift amend <slug> --entries <file>` carrying `metadata` marks as the way forward.
  They name no such cell to the writer.
- A cell is unchangeable in two cases.
  One is a cell whose old value names the baseline's commit and whose new value names the code HEAD.
  The other is a cell a second current capture of the same code gives another value or none.

## Design decisions to review

- Lym hard-codes no provenance name such as `_source.txt`; the expect agent marks what it finds, so the marks live in the run, not in a hub's `drift.yaml`.
- Commit detection takes a hex run of at least `minCommitAbbrev` digits, whole or abbreviated, as a naming of a commit.
- The second capture of the same code runs only when a cell is left after the commit rule, and one that does not pass marks nothing.
- The `diff: command` mode marks whole cases, since it has no line keys.

## To check or run by hand

- No regression check ran for this task.
- The Models hub's capture harness is outside this repository and still writes `_source.txt` into `{baseline_dir}`.
  It must move that file to `{meta_dir}`, or the expect agent marks it as metadata on each run.
  Neither this change nor the Models hub's `drift.yaml` needs a `volatile_files` entry for it.
- The capture record already holds the HEAD a capture ran at, and the landing's `## Regression check` should name that commit and the metadata marks.
  Check on the first landing of a drift run that both appear there.
