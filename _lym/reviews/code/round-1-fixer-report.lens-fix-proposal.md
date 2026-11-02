```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer applied as proposed (F1–F5) and build a counter-case for each.

**LOW. F1, `internal/output/summary.go`, doc comment of `Summary`.**
The `Summary` doc still says "a list of plain values joined on one line, or its count when it is longer than the list limit". F1 changed that for a list under a note key. A `warnings` list of two items now prints one line per value, where it used to print `warnings: a, b`. A `warnings` list over five items no longer collapses at all. The `list` comment was updated. The `Summary` comment, which is the public contract, was not, and it now contradicts the code for note keys.

No counter-case holds for the rest:

- **F1 behaviour.** A plain list under a note key now goes through `noteLines`. A list whose items are all empty strings prints nothing and sets no `hid`. Earlier it printed `warnings: , `. That is not worse. Mixed lists that hold an object take the object branch, so they are unchanged. The `hid` flag is no longer set for a long note list, which is right because nothing is hidden.
- **F2.** The expected string `{"error":"boom","ok":false}` depends on `output.Err` marshalling a map in sorted key order. The first gate run showed that order, so it is stable.
- **F3.** Each new row's expected output follows from the code as read, and the gate passed.
- **F4.** Documentation only, with nothing to break.
- **F5.** `shedverbs` now uses `term.IsTerminal` in place of the `ModeCharDevice` stat. Output redirected to `/dev/null` or another character device used to print the human `lym shed status` view and now prints the JSON envelope. That matches the intended terminal rule. The tests override the `writerIsTerminal` variable, so none depends on the old mechanism. `shedverbs` already imported `clihelp`, so no import cycle arises.

Further tool calls: none.
