# selfreport create: add --body-file and refuse -b text naming a file

An agent filed issue #136 with `lym selfreport create "<title>" -b .scratch/selfreport-missing-brief.md`.
`-b` takes the body as text, so the issue's body became the path and the report in the file was lost.

## What changes

- `lym selfreport create` gains `--body-file <path>`, which files the file's contents as the issue body.
  `--body-file -` reads the body from stdin, as `-b -` already does.
- `--body` and `--body-file` are mutually exclusive; cobra refuses both together.
- `--body` text that, trimmed, is one line naming an existing regular file is refused before any GitHub call.
  The error names the way forward, `pass --body-file <path> to file the file's contents`.
  Text that mentions a path inside a longer line, or names a directory or a missing file, is still filed as text.
- A `--body-file` that cannot be read fails with `failed to read --body-file: ...` and files nothing.
- The command's `--help` examples show `--body-file` and the refusal, since the friction reflection and agents learn the flags from it.
  `docs/overview.md`'s selfreport entry and the package doc say the same.

## Design decisions

- The guard refuses rather than warns: a warning on a JSON envelope that still reports success would let an agent file the path and move on,
  which is the failure the task reports.
- A relative path is resolved against the process's working directory, the same directory the caller's shell resolves it in,
  so the check sees the file the caller meant.

## Tests

`internal/selfreportcli/cli_test.go` adds rows to the existing tables:
a body from a file, a body file read from stdin, and body text that mentions a file inside a longer line succeed with the expected request body;
`-b` naming an existing file, both body flags together, and a missing body file are refused with no request sent.
The rows run in a temp directory holding the report file, entered with `t.Chdir`.
