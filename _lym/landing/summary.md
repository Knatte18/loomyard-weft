# Operator verbs print a short readable summary on a terminal

An operator verb such as `lym adopt root` printed one long JSON envelope.
On a stop it carried every commit hash of a divergence and every run's record, so the operator could not see what happened or what to do next.

On a terminal, a command that carries the `operator` audience now prints each envelope line as a short summary:
the outcome, the fields, and every error, warning and way forward, one line each.
Long lists collapse to a count, and the summary then ends with `full envelope: rerun with --envelope`.

The global `--envelope` flag keeps the full JSON where the summary would print.
Output that is not a terminal, as an agent's or a script's, always gets the envelope, so no stencil or skill needs the flag.
This settles the entry's open question about agents calling these verbs.

## Design

- `output.Summary` renders one envelope line.
  A line without a boolean `ok` key is not an envelope, and `Summary` reports false for it.
- `output.SummaryWriter` filters the command's output stream.
  It holds a line that opens with `{` until its line break, renders it when it holds an envelope, and passes every other byte through.
  Prompts and other text therefore reach the terminal at once.
- A plain list collapses to a count past five values, and an object nested deeper than two levels collapses to its field count.
  A list of objects never collapses; each item is one line with its label and line field.
- Every error, warning and `way_forward` value prints one line each at any depth, including inside what a count hides.
  A plain list under such a key never collapses.
- `clihelp.UseReadableOutput` makes the choice in the root's pre-run.
  It skips a command that is not an operator verb, one with `--envelope` set, and one with its own local `--json` flag, such as `lym shed status --json`, which renders its own terminal view.
- A command's own local `--envelope` flag, such as `lym creel read`, shadows the global one and keeps the envelope when set.
- `clihelp.RunRootCtx` flushes the writer when execution ends, before any cobra error envelope is written, so a held unterminated line comes out first.
- `shedverbs` now builds `writerIsTerminal` from the shared `clihelp.WriterIsTerminal`, which checks for a terminal descriptor, in place of its own copy.
- `docs/overview.md`, `pattern/PATTERN-cli-cobra.md` and the root help text in `cmd/lym/main.go` describe the flag.

## To check by hand

- Run `lym adopt root` against a stop on a real terminal, and confirm each stop reads as one line with its way forward and that `--envelope` prints the full JSON.
- Confirm the same run piped to a file prints the envelope.
- The summary of a list under an error, warning or way-forward key in `internal/output/summary.go` (`summaryRenderer.list`) was once collapsed to a count and now prints one line per value; confirm a stop with more than five warnings or ways forward shows them all.
