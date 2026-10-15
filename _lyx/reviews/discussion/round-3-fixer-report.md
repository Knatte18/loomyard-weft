# Fixer report — discussion, round 3

## Changes

- **F1** (fixed as proposed): the first Constraints bullet now quotes the amended `PATTERN-lyxdirs-single-declarer` line, scoped to Go paths and Go production files with the shell-tool clause appended.
  The separator after "in a Go path" is a colon instead of the proposed dash, matching the current `PATTERN.md:9` line's form.
  The `-weft` sentence and the Scope In line are unchanged.
- **F2** (fixed as proposed): D1 now says the report write reaches exactly two paths, `--out` and one `mktemp` file in `--out`'s directory that is removed on failure.
  It also says nothing outside `--work` is otherwise created, replaced or deleted, and that `--out`'s bound is the explicit flag and the caller's permissions, as with a shell redirect.
  `mktemp` is added to the required commands.
  The failed-run acceptance criterion now also requires no temporary file left beside `--out`.
- **F3** (fixed as proposed): D1's third clearing bullet now says all its comparisons use canonical paths (`realpath -m`), and that the marker rule is the bound if a comparison is ever wrong.
  `realpath` is added to the required commands.
  A new acceptance criterion covers a `--work` given as a symlink to the repo root and one spelled with `..` to reach `--out`'s directory: each exits 1 naming `--work` and deletes nothing.

`support-log.md` is unchanged.

## Disputed

None.

## Deferred

None.
