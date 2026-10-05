---
round: 1
focus:
  - "Card 1, `internal/shuttleengine/claudeengine/command.go` and `claudeengine.go`: does the `CLAUDE_ENV_FILE` assignment lead both the launch and the resume line on posix and pwsh, go through `internal/shell` `WithEnv` only, and keep the line single-line? Does `Prepare` pass the `filepath.Abs` path to both builders?"
  - "Card 1, `settings.go`: after the stdin hook removal, is the webster fork guard the only Bash PreToolUse entry, and do the Agent, AskUserQuestion and Stop hooks stay unchanged? Do any dead references to the deleted `bashStdin*` identifiers remain in code, docs or comments?"
  - "Card 1 tests: do the two deleted test files go with no dangling references, and does `envfile_integration_test.go` carry its `integration` build tag with the three rows the card names (bare `cat`, heredoc, pipe)?"
  - "Card 2, `internal/shedverbs/step.go` and `steprecord.go`: on every exit path (success and each refusal), is the full envelope written to the record before stdout is printed? Do the short and full outputs and the exit code match the card in all four cases (record written or not, `--full` or not)? Does the short line ever name a missing file?"
  - "Card 2: are the two short key sets (success, error) built in one helper per envelope kind with the closed-set comments, and does `envelope_path` come only from the recorder's returned path built from `Spec.StepsDir`? Are the `StepEnvelope` and `stepErrFields` full key sets unchanged?"
  - "Card 2, callers of the old stdout keys: does anything outside the updated tests still parse `friction_dir` or `scratch_dir` from step stdout, including `internal/loomcli/cli.go` `Long` text, `pattern/PATTERN-shed-verb-set.md`, `docs/overview.md`, and any test asserting the old `Long` sentences?"
  - "Card 2 tests in `steprecord_test.go` and `internal/shedcli/parity_test.go`: does the table cover success and refusal, each without and with `--full`, plus the `last_step.envelope_path` equality? Are the replaced tests free of leftovers?"
  - "Card 3, `contracts/stencils/shed/shed-template-driver.md`: does the invocation block hold exactly `lyx shed step {{.run_id}}`, is the `(cd <drive-dir> && lyx` form gone, and do `## Error envelopes`, `## Parking` and `## Repair records and the stop report` each carry their own `envelope_path` sentence? Does the stencil stay recipe-blind, ask no operator, keep one sentence per line, and point at `contracts/specs/refusal-spec.md` without restating it?"
  - "Doc comments and docs landing in the same commit: `doc.go` in `claudeengine` and `shedverbs` match the new behavior, and new or changed comments follow the repository's `CLAUDE.md` and `PATTERN.md` conventions."
  - "Test economy: does any new test duplicate an existing test's coverage, or does any added behavior lack coverage? Name the covering test for any redundancy finding."
---

Range: no `_lyx/loom/rework/` exists, so there is no live round.
The review range is `git diff $(git merge-base main HEAD)..HEAD`, with `product.parent` read as `main` from `_lyx/shed/agent-command-wrapping/status.json`.
Each entry names where to look and the question to settle; none caps severity or states a verdict.
