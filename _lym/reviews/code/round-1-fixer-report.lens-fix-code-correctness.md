```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran each changed branch (F1–F5) on a concrete input and checked callers and whether each added test would fail with the defect in place.

**LOW — `internal/output/summary.go`, the `Summary` doc comment**
- The comment still says "a list of plain values joined on one line, or its count when it is longer than the list limit".
- The new branch in `list` breaks that for a note key such as `warnings`, whose plain list now prints one line per non-empty value at any length, even three or five items.
- Only the `list` comment was updated, so a reader of `Summary` gets the old rule.
- Run on `{"ok":true,"warnings":["a","b"]}`: it prints `warnings: a` and `warnings: b`, not `warnings: a, b`.
- The review's fix named only the over-five case. Applying it to every length is consistent and matches the entry's "one line each", so this is a stale sibling comment, not a wrong behaviour.

Everything else I traced holds:
- **F1:** `{"warnings":["w1",…,"w6"]}` goes `field` → `list` → `noteLines` and prints six `warnings: wN` lines. The `LongNoteList_OneLinePerWarning` row would fail with the old collapse, which prints `warnings: 6 items`.
- **F1, boundary row:** `FivePlainItems_JoinedWithoutFooter` fails if `>` becomes `>=`.
- **F1, `NotesOutsideCollapsedObject_EachOnOwnLine`:** it passes against the pre-fix code too, since depth 0 and 1 notes already printed through `field` and `text`. It covers a path the review said had no row, but it is not a regression guard for F1.
- **F2:** with `failing` the writer holds the unterminated `{"ok":true}`, and `RunRootCtx` runs the finish steps before `output.Err`. The result is `ok\n` then the error envelope, exit 1. If the flush ran after the error envelope, the order would flip and the row would fail.
- **F2, key order:** after the gate's first failure the expected string is `{"error":"boom","ok":false}`, and the second gate run was green.
- **F2, local flag:** the `localenvelope` row prints the envelope because `cmd.Flags().Lookup("envelope")` finds the local flag and `Changed` is true. Removing the lookup makes it print the summary and fail.
- **F5:** `var writerIsTerminal = clihelp.WriterIsTerminal` keeps the type `func(io.Writer) bool`. The only caller is `status.go:218`, and the only test override is `human_test.go:154-156`, which still assigns a compatible closure.
- **F5, behaviour change:** a writer that is a char device but not a terminal, such as `/dev/null`, used to count as a terminal for `lym shed status` and no longer does. That is the intended single check.
- **F5, imports:** `shedverbs` already imported `clihelp`, so there is no cycle, and removing `io` and `os` from `human.go` is clean.
- **Other tests:** no test elsewhere depends on a joined `warnings:` line. `output.Summary` and `NewSummaryWriter` have no callers outside `internal/output` and `UseReadableOutput`.

Tool calls:
1. One grep for `output.Summary`/`NewSummaryWriter` callers, every `writerIsTerminal` use, and tests asserting `warnings: `/`way_forward: `. My context lacked the callers outside the files read.
