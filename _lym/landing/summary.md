# selfreport create: add --body-file and refuse -b text naming a file

An agent filed issue #136 with `lym selfreport create "<title>" -b .scratch/selfreport-missing-brief.md`.
`-b` takes the body text, so the issue body became the path and the report the file held was lost.

`lym selfreport create` now has two changes:

- A `--body-file <path>` flag files the contents of a file as the issue body; `-` reads stdin, as `-b -` does.
  It is mutually exclusive with `-b`/`--body`.
- `-b` text that is one line naming an existing regular file is refused.
  The error names `--body-file` with the path as the way forward.
  Surrounding whitespace is trimmed before the check.
  A directory, a multi-line text, a line that only mentions a file, and an empty body are filed as text as before.

`docs/overview.md` and the command help carry the new flag and the refusal.

Closes #136

## Design decisions

- The check refuses rather than warns, because a warning still files the wrong body and the report cannot be recovered afterwards.
- Only regular files trigger the refusal, so `-b .` and similar text are not blocked.
- Body resolution moved from `runCreate` into `resolveBody` and `readBodySource` in `internal/selfreportcli/cli.go`.
  The `nil` body (neither flag given) and the empty body (`-b ""`) keep their earlier meaning.

## To check

- The refusal is only as good as `os.Stat` from the working directory: text naming a file that exists elsewhere is filed as text.
- Check `body-flag-names-raw-literals`: the flag names are now the constants `bodyFlag` and `bodyFileFlag`, and `stdinArgument` names `-`.
  Confirm no raw `"body"`, `"body-file"` or `"-"` literal remains in `internal/selfreportcli/cli.go`.
- Check `padded-path-trim-untested`: a test row now covers padded text (`" report.md\n"`) refused with the trimmed path in the message.
- Not run by hand: an end-to-end `lym selfreport create` against GitHub; the tests use an `httptest` server.
