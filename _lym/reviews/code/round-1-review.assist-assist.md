---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/selfreportcli/cli_test.go:176
    summary: New filename-guard boundaries and empty-file bodies lack public-surface coverage
    fix: Extend the existing CLI success and refusal tables in internal/selfreportcli/cli_test.go. Add a refusal row passing an existing relative filename padded with spaces and a trailing newline, asserting exit 1 and no request. Add success rows passing a directory name as body text and an empty file through --body-file, asserting the unchanged directory text and a present, empty body field respectively. Extend the existing fixture to supply the directory and empty file; keep the rows in the existing tables.
    origin: lens:code-tests
---

### [BLOCKING] Cover the new body-source boundaries

**Location:** internal/selfreportcli/cli_test.go:176; internal/selfreportcli/cli_test.go:301

**Issue:** The tables cover a nonempty file and an unpadded regular filename, but omit distinct new behavior.
`existingFileNamedBy` trims whitespace at `internal/selfreportcli/cli.go:166` and excludes directories at line 171.
Removing either operation would leave the current rows green while changing the promised behavior.
The file reader also returns an explicit empty body at lines 159–160, which the nonempty fixture does not pin.
The rubric requires findings for uncovered new behavior.
`PATTERN-test-speed` incorporates the testing skills' boundary-coverage requirement;
the round instructions classify violations of listed constraints as BLOCKING.
This is a coverage finding, not an observed implementation failure.

## Review evidence

Reviewed the supplied diff pieces and commit messages for `22a2c25b767617b80815edf3f0748f816ac96796..HEAD`.
The board entry and decision record ask for the file flag, stdin support, and refusal or warning for filename bodies.
The implementation delivers those changes without unrelated scope.

The focus cases were traced through the command, shared execution wrapper, output envelope, and issue engine.
Relative paths, trailing whitespace, symlinks, and paths outside cwd follow the intended regular-file check.
Ordinary one-line text is preserved unless its trimmed entirety names an existing regular file.
Both stdin flags preserve their input, and explicit empty body text and empty files produce non-nil body pointers.
Directories and missing files remain literal --body text; a directory given to --body-file returns a read error.
Mutually exclusive flags return through the shared error envelope with exit 1 before transport.
Serial subtests and cleanup keep cwd and package-level seams isolated.
The new table rows do not duplicate existing flag behavior.
The help examples, overview entry, and landing summary describe the implemented interface.

All lens reports were saved verbatim beside this report.
No lens findings were rejected.

## Validation limit

Attempted `timeout 3600s lym gate test ./internal/selfreportcli --tier 0`.
The gate could not acquire a slot because its holder records under the hub's `_bolt/.lym/gate` were read-only.
No package tests ran in this review attempt, and that infrastructure failure is not a code finding.
