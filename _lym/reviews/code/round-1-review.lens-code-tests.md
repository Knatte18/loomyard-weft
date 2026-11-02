```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that pins it, or none.

**Per-behaviour list**

Summary rendering (`internal/output/summary.go`)
- `Summary` returns false for a non-envelope line (plain text, no `ok`, array, two envelopes): `TestSummary` rows PlainText, ObjectWithoutOk, Array, TwoEnvelopes.
- `ok` opens the summary, and empty or false fields are left out: `OkScalars_EmptyLeftOut`.
- `failed: <error>` with a multi-line error indented: `Failed_MultiLineErrorIndented`.
- A plain list over five items collapses to a count and adds the footer: `LongPlainList_CollapsesToCount`. A short list on one line is pinned by `tags: a, b` in the same row.
- Exactly five items stays uncollapsed: none. The row uses 6 and 2, so a `>` changed to `>=` would still pass on the 6 row but not on a 5 case, and no row sits on the boundary.
- An object list prints one line per item as label plus line field, and a `way_forward` note prints under the item: `ObjectList_OneLinePerItemWithNotes`.
- An item with a label and no line field, or a line field and no label: none. Only both-present and neither are covered.
- An item with no label or line key falls back to `key=value` plain fields: `ObjectListWithoutLabel_PlainFields`.
- The footer appears when an item hides a non-note field (the `hid` loop in `item`): row `ObjectList_OneLinePerItemWithNotes` gets it from the `state` field of item a. `ObjectListWithoutLabel_PlainFields` pins the opposite case, where nothing is hidden.
- A scalar plain-list item that is not an object in an object list (`- value` branch): none.
- An object nested past depth two collapses to a field count, and the notes inside it keep their path: `DeepObject_CollapsesKeepingLabelledNotes`. The path covers `tips.t.error` and `warnings` as a list.
- An error or warning at depth zero or one inside a non-collapsed object: none directly. Only the deep row exercises `notes`, and the `_error` suffix key, a scalar `*_warning`, and a `way_forward` list are not covered there.
- `{}` and `null` fields left out: none, though `isEmpty` is partly covered by the empty string and false in row 1.
- A number printed from `UseNumber`, such as the `count: 3` check: covered by `OkScalars_EmptyLeftOut`.

Stream filter (`internal/output/summarywriter.go`)
- An envelope split across writes, with text around it, and a prompt with no newline passing through at once: `EnvelopeSplitAcrossWrites_TextAround`. The `wantBefore` check pins the immediate pass-through only for the `prompt: ` part.
- A `{` line that is not an envelope is written as written, and a `{` in mid-line is not held: `BraceLineNoEnvelope_AsWritten`.
- A line that never ends is held until `Flush`: `UnterminatedEnvelope_HeldUntilFlush`.
- Write byte counts and error returns on a failing underlying writer: none. Every `Write` assertion expects a nil error and a full count, and no row uses a failing writer.
- Two envelope lines in one write: none. Each row has at most one envelope per write.
- `Flush` with nothing held is a no-op: none, although it is trivial.
- `Flush` with a held non-envelope line writes it as written: none. Row 2 ends with newlines, so nothing is held at `Flush`.

Wiring (`internal/clihelp/readable.go`, `exec.go`, `main.go`)
- An operator verb on a terminal gets the summary: `OperatorOnTerminal_Summary`.
- `--envelope` keeps the envelope: `EnvelopeFlag_Envelope`.
- The role audience keeps it: `RoleAudience_Envelope`.
- A non-terminal output keeps it: `NoTerminal_Envelope`.
- A command with its own `--json` keeps it: `OwnJSONFlag_Envelope`.
- The global `--json` is not mistaken for a command's own (the `*jsonHelpValue` check). No row passes a case where the global `--json` flag is present on an operator verb and the summary still prints, except incidentally: every operator row inherits the global flag and `OperatorOnTerminal_Summary` would fail if that check broke. So it is pinned.
- `onFinish` flushes in `RunRootCtx` before the cobra error envelope, and runs when `RunE` succeeds: `UnterminatedEnvelope_SummaryAtEnd`. This covers only the success path.
- The flush ordering when cobra returns an error (a held line flushed before the error envelope is written): none. No row has a failing `RunE` or a bad flag with a held line.
- A summary on an error path: a verb that returns an error envelope on a terminal. No row; the only envelopes tested are `ok:true`.
- `onFinish` with a nil exit state is a no-op: none. This is trivially unreachable from `RunRootCtx`, so it is low value.
- `WriterIsTerminal` for a non-`*os.File` or a file that is not a terminal: none. It can't be driven without a tty, and the tests inject a fake `isTerminal`.
- A command's own `--envelope` flag shadowing the global one, as `lym creel` and `lym loom arbiter` do (the `Changed` check on a local flag): none. All rows use the global flag, and the summary claims the behaviour in its Wiring section.
- `--envelope` appearing in the JSON help and the command index: none. No test in the diff touches `jsonhelp` or `index` output.
- `cmd/lym/main.go` wiring, with `UseReadableOutput` called in the root's `PersistentPreRunE` and `InstallReadableOutput` on the root: none. `readableRoot` rebuilds the wiring by hand, so a deletion of those lines in `main.go` would pass every test. There is no test that builds the real root.

**Findings**

1. LOW, design, `internal/clihelp/readable_test.go:53` (`TestUseReadableOutput`). There is no row for the flush ordering on a cobra error: an operator verb that holds an unterminated line and then fails. The behaviour the diff changed in `exec.go:194-198` (finish steps run before the error envelope) has only a success-path row. Add a row with a verb that writes an unterminated `{"ok":true}` and returns an error, and assert the summary precedes the cobra error envelope.

2. LOW, design, `internal/clihelp/readable_test.go:53`. There is no row for a command's own `--envelope` flag shadowing the global one (the `lym creel` and `lym loom arbiter` case, `readable.go:42-44`). Add a row where the verb declares a local `--envelope` flag and assert that setting it keeps the envelope.

3. LOW, design, `internal/output/summarywriter_test.go:12`. No row covers an error from the underlying writer, so the returned byte count `total - len(p)` and the error path in `Write` are unpinned. Add a failing-writer row that checks the count and the error for both a pass-through chunk and a held envelope line.

4. LOW, design, `internal/output/summary_test.go:33`. The collapse threshold has no boundary row: exactly five plain items should print on one line, and the test uses 6 and 2. Add a five-item list to `LongPlainList_CollapsesToCount` and assert it is joined on one line with no footer.

5. LOW, design, `internal/output/summary_test.go:55`. The note paths have thin coverage: a note at depth zero or one inside a non-collapsed object, the `*_error` suffix key, a scalar `*_warning` and a `way_forward` list are not pinned. Add a row with a failed envelope carrying a nested object with a `way_forward` and an `x_warning`.

6. LOW, consistency, `cmd/lym/main.go:170,176`. Nothing tests that the real root calls `UseReadableOutput` and `InstallReadableOutput`, because the clihelp test builds its own root. Add a test that builds the root `cmd/lym` constructs and asserts `--envelope` is declared, or accept the gap.

**Tool calls:** none made. Everything came from the inherited context.
