# lyx board upsert reads the body from a file or stdin

`board-body-file` · Someday · feature

Seen 2026-10-03: `lyx board upsert` takes the whole body inside one JSON argument, so changing one section of an entry means reading the body, splicing it and re-encoding JSON in a shell one-liner, which breaks on quoting (an apostrophe in the text) and pushed the orch into a throwaway script.
Idea: `upsert` accepts the body from a file or stdin (for example `--body-file`), so the orch writes the body with its file tools and sends it unchanged.
