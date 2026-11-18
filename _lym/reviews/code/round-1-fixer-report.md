---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

## Changes

All four findings were fixed as proposed, in one commit, `c51c73a6`.
The edits to `cli.go` and `cli_test.go` interleave across findings, so one commit per finding was not separable.
The package gate `lym gate test ./internal/selfreportcli` passed.

- F1: `internal/selfreportcli/cli.go` declares `bodyFlag` and `bodyFileFlag` beside `stdinArgument`, and every flag-name literal in `Command` and `resolveBody` uses them.
- F2: `TestCommand_RefusedBeforeTransport` gains the row "body text naming an existing file with padding".
- F3: `TestRunCreate_Success` gains the rows "body text naming a directory" and "empty body via flag".
- F4: the refusal formats the path with `%q`.
  The test rows expect the JSON-escaped form `pass --body-file \"report.md\"`, because the envelope carries the message as JSON.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
