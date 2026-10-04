# Card 6 — board-upsert-body-file

**Edit:**
- `internal/boardcli/cli.go`
- `internal/boardcli/verbs_test.go`

**Create:**
- `internal/boardcli/bodyfile.go`
- `internal/boardcli/bodyfile_test.go`

**Intent:**
D8: `lyx board upsert` takes `--body-file <path>`, where `-` reads stdin, and the content becomes the payload's `body`.

- `bodyfile.go` holds one helper that, given the decoded payload, the flag value, the raw payload argument and a stdin reader, sets `fields["body"]` from the file or stdin.
  It refuses a payload that already carries `body`, naming both sources (drop `body` from the payload, or drop the flag);
  and it refuses `--body-file -` when the payload argument is itself `-`, since stdin can feed only one of them.
  A read failure is returned naming the path.
- `cli.go` registers the flag on `upsert` only, reads stdin through the command's `InOrStdin` so tests can feed it, calls the helper before `UpsertTask`, and documents the flag and both refusals in the help.

Bound: the flag only supplies `body`; every other field still comes from the JSON payload and passes the same validation.
It reads exactly the named path or stdin with the caller's own permissions.

Tests: `bodyfile_test.go` (untagged, temp files only) covers a file, stdin, an empty file, a missing file, `body` plus the flag, and `-` for both payload and flag;
`verbs_test.go` drives `upsert --body-file <path>` and `--body-file -` end to end and reads the body back with `get`.

**ImpactSummary:** Adds one flag to `upsert`; an upsert without the flag behaves as before.
