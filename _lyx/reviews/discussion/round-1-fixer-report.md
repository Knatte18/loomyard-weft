# Round 1 fixer report: discussion

Every finding was checked against the code and fixed; none was disputed or deferred.

## Fixed

- **F1** (D6, acceptance criteria): the form test splits `Use` on whitespace except that a brace-balanced `{…}` span is one token, and it checks only that the braces balance; D7's coupling test owns the keys.
  The acceptance criterion lists `{…}` among the accepted forms and adds an unbalanced-brace failure row.
- **F2** (D7, Open risks): confirmed that `merge`'s parser refuses a payload without `upsert` (`internal/boardcli/cli.go`), so the token now reads `upsert: <task>`.
  The coupling test compares required keys, optional keys and exclusive pairs, nested objects included, against one declaration per parser that the parser also validates with.
  The merge Open risk is dropped and ledger Q12 is resolved.
- **F3** (D4): confirmed that `lyx orch watch` builds the watcher once (`internal/orchcli/watch.go`) and nothing restarts it on a deploy.
  D4 names the binary the index describes, the `lyx` on the orch's `PATH`.
  Start, `--adopt` and `resume-context` render in-process as fresh processes of that binary, and the watcher takes an index source from `orchcli` that runs `lyx help index` at each render.
  This adds a Constraints line (`PATTERN-spawn-observability`, `PATTERN-told-geometry`), auto-pick A9, ledger Q14, two rejected alternatives and an acceptance clause.
- **F4** (D2, D3): confirmed `config [module]` with `--print` and `--set` (`internal/configcli/configcli.go`).
  D3 now defines an invocable command as a leaf or a runnable group, and the audience rules and test cover both.
  D2 renders a runnable group's line in the leaf form, and the acceptance criteria add rows for `lyx config` and the runnable-group test case.
- **F5** (D2): the index skips `completion` and never prints a command without an `audience`.
  An acceptance row covers this.
- **F6** (D1): confirmed cobra v1.10.2's default help carries a `ValidArgsFunction` and the "Unknown help topic" fallback.
  D1 gains a bound: argument completion is kept, `lyx help <command>` is routed, and an unknown topic gets a JSON error.
  An acceptance row covers this.
- **F7** (D11): a comment is dropped only when its whole trimmed body equals the pointer form with any slug.
  The import acceptance criterion adds a kept-quoting-comment row.
- **F8** (D1, Open risks): decided that `lyx --json` lists `help` with `index` and keeps skipping `completion`.
  The Open risk is dropped, ledger Q11 is resolved, and an acceptance row covers this.
- **F9**: the D2 and D9 measurements moved to the support log's Interview turn 7 and the full-path rejected alternative.
  D6's tally now reads "every command that spells an optional flag in `Use`", and the registration route stays in Constraints only.
  The support log's Review rounds section records the round.

## Disputed

None.

## Deferred

None.
