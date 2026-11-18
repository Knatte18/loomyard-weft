# selfreport -b takes a file path as the body text, losing the report

`selfreport-body-path-as-text` · Task · bug, selfreport · draft

An agent filed issue #136 with `lym selfreport create "<title>" -b .scratch/selfreport-missing-brief.md`.
`-b` takes the body text, so the issue body became the path and the report itself was lost.

Idea: give `lym selfreport create` a `--body-file <path>` flag (with `-` for stdin), and refuse or warn when `-b` text is a single line naming an existing file.
