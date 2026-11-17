```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test row that would fail if it broke.

**Per behaviour, the test that pins it**

- **The hook hashes the content of a readable file into `content_hash`.** `TestRecordEdit/AnEditOfAReadableFileCarriesItsContentHash`, at `internal/shuttleengine/claudeengine/editrecord_test.go:47-50`.
- **An unreadable file is recorded with no hash.** `TestRecordEdit/AnEditOfTheSessionIsAFileEdit`, where `/w/a.go` does not exist, at `editrecord_test.go:52-56`.
- **`io.Copy` fails after the open succeeds, for example on a directory.** No row. This is trivial and I file nothing on it.
- **`fileEditSignal` reads `content_hash` into `SessionSignal.ContentHash`.** The same readable row, through `ParseSessionSignals` and `assertSignals`, at `editrecord_test.go:117-121`.
- **The record carries no edit text and no file content.** The `carried` loop at `editrecord_test.go:109-113`.
- **`fold` skips a signal with an empty `ContentHash`.** `TestWait_EditLoopNotice/"edits whose file the hook could not read never count"`, at `editloop_test.go:589`. Without the skip, four equal empty hashes would make three repeats and send a notice.
- **An edit counts as a repeat when its hash matches any earlier content, not only the previous one.** The `flip` row, h1 h2 h1 h2, at `editloop_test.go:580-583`.
- **Edits that make progress never count.** The row "edits that each leave new content…", at `editloop_test.go:585-588`.
- **Agents and files keep separate histories.** The rows "the session's and a subagent's edits…" and "edits of two files count apart", at `editloop_test.go:600-601`. Their hashes are all h1, so a merged key would notify.
- **The subagent is named in the notice.** The row "a subagent's loop names the subagent", at `editloop_test.go:594-597`.
- **Content older than the window is pruned.** The row "content older than the window…", at `editloop_test.go:613-615`. At the step-12 edit, 65 s in, h2@0 is gone; without the pruning it would be a second repeat and notify.
- **A repeat older than the window is pruned** (the `history.repeats = slices.DeleteFunc(...)` line, `editloop.go:471`). **None.** See finding 1.
- **The whole history is cleared after a notice.** The row "a loop twice as long notifies twice", at `editloop_test.go:638-641`. Clearing only the repeats would give three notices instead of two.
- **A signal with no `At` takes `now`, and the stamp is honoured.** The cross-tick rows and the stamped rows, at `editloop_test.go:606-633`.
- **`clone` is deep and the watch is restored on a failed save.** The row "a failed offset save is retried…", at `editloop_test.go:652-653`.
- **The notice carries the repeat count.** The `"%d times"` assertion, at `editloop_test.go:679-680`.
- **The template default of 3.** `TestLoadConfig_EditLoop/TemplateValues`, at `config_test.go:204`.
- **`DropEditLoopEdits` drops a value of 0, drops any count and keeps the other keys.** `TestDropEditLoopEdits`, rows AZeroCountIsDropped and AnyCountIsDroppedAndOtherKeysKept, at `reinterpret_test.go:791-797`.
- **A file without the key, or an empty file, comes back unchanged.** Rows AFileWithoutTheKeyIsUnchanged and AnEmptyFileIsUnchanged, at `reinterpret_test.go:798-799`.
- **A file that does not parse is an `ErrInvalid` error.** Row AnUnparsableFileIsInvalid, at `reinterpret_test.go:800`.
- **A null or empty `edit_loop_edits`, and a document whose top level is not a mapping.** No row. See finding 2.
- **The shuttle module registers `EditLoopRepeatsReinterpretation`.** `TestModules_RegisterTheirReinterpretation/shuttle`, at `configreg_test.go:69-70`.
- **The reinterpretation applies once, behind its marker.** This is the generic behaviour of `hubreconcile`, which the diff does not change; that package's own tests cover it.

**Findings**

1. **MEDIUM, design. No row pins that a repeat expires out of the window.**
   - Location: `internal/shuttleengine/editloop.go:471`. Evidence: `internal/shuttleengine/editloop_test.go:613-620`.
   - Deleting the `history.repeats` pruning line fails no row of `TestWait_EditLoopNotice`.
   - The window rows either never produce a repeat (the stamped 0/120/240/360 row), or produce only one, which stays below the count with or without the pruning (the step-5/12 row).
   - Without the pruning, two short loops further apart than `edit_loop_edits`' window would add up to one false notice.
   - Fix: add a row in which repeats land, more than the window passes, and further repeats land that alone stay below the count. One example with the count at 2: stamped h1@0, h2@1, h1@2 (1 repeat), then h3@100, h4@101, h3@102 (1 repeat). This row wants no notice.

2. **LOW, design. The null or empty value, and a non-mapping document, have no row.**
   - Location: `internal/shuttleengine/reinterpret.go:743-747`. Evidence: `internal/shuttleengine/reinterpret_test.go:791-801`.
   - The focus asks how a scalar, null or missing `edit_loop_edits` is handled. `edit_loop_edits:` or `edit_loop_edits: null` decodes to a non-zero `yaml.Node`, so it is dropped with a line that names an empty value or `null`. No test pins this.
   - A document whose top level is a sequence is also unpinned. It fails inside `yaml.Unmarshal` into the struct, and that error is marked invalid.
   - Fix: add rows for `edit_loop_edits:` (null) and for `- a\n` (non-mapping, `wantErr`).

3. **NIT, consistency. One string in the "carried" list can never appear.**
   - Location and evidence: `internal/shuttleengine/claudeengine/editrecord_test.go:109`.
   - No payload or written file in `TestRecordEdit` contains `"package b"`, so that assertion can never fail.
   - Fix: drop `"package b"` from the list.

Tool calls:
- `grep` for `dialogTick` and a read of `editloop_test.go:130-200`, to get the tick length and the harness's time base, which I needed to work out the window-pruning rows. My context held only the diff hunks of that test.
