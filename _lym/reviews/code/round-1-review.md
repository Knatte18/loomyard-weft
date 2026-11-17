---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/shuttleengine/reinterpret.go#DropEditLoopEdits
    summary: The reinterpretation drops a non-zero edit_loop_edits, which neither rewrites the value to its old meaning nor flags it, as PATTERN-config-layers requires.
    fix: >-
      In DropEditLoopEdits, drop the key only when its value turns the watch off: the integer 0, or a null or empty value, which loads as 0.
      Report that drop in the rewrote line as now.
      For any other value, return existing unchanged and put one line in flagged instead of rewrote.
      The line names the value, says edit_loop_edits now counts only edits that return a file to earlier content, so the value means a different count,
      and gives the way forward: lym config shuttle --layer default --set edit_loop_edits=<count> from the prime worktree, or drop the key to take the template's count.
      Update the function's doc comment, the reinterpret.go file header, the matching lines in internal/shuttleengine/doc.go, and TestDropEditLoopEdits.
      In the test, AnyCountIsDroppedAndOtherKeysKept becomes a row that wants the file unchanged and one flagged line naming 6;
      the rewrote and flagged checks then assert per row which list holds the line.
      Update _lym/landing/summary.md's reinterpretation bullet and its last design decision to match.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/editloop.go:471
    summary: No test row fails when the pruning of history.repeats is removed, so a repeat expiring out of the window is unpinned.
    fix: >-
      Add a row to TestWait_EditLoopNotice in internal/shuttleengine/editloop_test.go, with markEvents true and the count of 2 the test uses.
      Its events are stamped(0, "h1") + stamped(1, "h2") + stamped(2, "h1"), one repeat,
      then stamped(100, "h3") + stamped(101, "h4") + stamped(102, "h3"), one repeat more than the one-minute window later.
      The row wants no loop.
      Name it for what it pins, for example: a repeat older than the window drops out of the count.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttlecli/hookedit.go:26
    summary: The hook-edit verb's Long help still says it records only the edited file and that the wait reports an agent that edits one file again and again.
    fix: >-
      In hookEditCmd's Long text, say the verb appends one record of the edited file and a hash of its content,
      and that the run's wait counts the edits that return a file to content it held earlier, to tell its parent about an agent that undoes or repeats its own edits.
      Keep the rest of the text.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/engine.go:314
    summary: The EditRecorder seam doc does not say a record carries the content hash, though the watch ignores every edit signal without one.
    fix: >-
      In the EditRecorder doc comment, add that each record carries a hash of the edited file's content right after the edit,
      and that an edit recorded with no hash never counts toward an edit loop.
      In the RecordEdit method doc, say it appends the file edit with that hash.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/reinterpret_test.go:791
    summary: TestDropEditLoopEdits has no row for a null or empty edit_loop_edits, nor for a document whose top level is not a mapping.
    fix: >-
      Add two rows to TestDropEditLoopEdits:
      one with existing edit_loop_edits:\n that wants the key dropped, want {}\n, and a line naming the drop, matching F1's handling of a null value;
      one with existing - a\n that wants an error marked configengine.ErrInvalid.
  - id: F6
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/reinterpret.go:753
    summary: The orch line prints the node's Value, which is empty for an empty, mapping or sequence value, so the mail reads edit_loop_edits  dropped.
    fix: >-
      When values.EditLoopEdits.Value is empty, name the value in the line as null for a null node and as its YAML kind otherwise,
      or describe it as the hub's value with no value printed.
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/editrecord_test.go:109
    summary: The carried list in TestRecordEdit checks for package b, which no payload or written file of the test contains, so that check can never fail.
    fix: Remove "package b" from the carried list in TestRecordEdit.
---

### [BLOCKING] The reinterpretation drops a non-zero count instead of flagging it

**Location:** `internal/shuttleengine/reinterpret.go#DropEditLoopEdits`

**Issue:** `pattern/PATTERN-config-layers.md` states the rule for changing a hub config value's meaning:
a binary carries a `configreg.Reinterpretation` that "rewrites the value to its old meaning, or flags it with a way forward".
`configreg`'s doc says the same: a rewrite "that keeps what the hub meant, or flags a value it cannot settle with a way forward".
A non-zero `edit_loop_edits` changes meaning in this diff, and the summary itself says a count of every edit has no equivalent count of repeats, so the value cannot be settled.
The rule's path for an unsettled value is the flag.
`DropEditLoopEdits` instead removes the hub's tuned value whatever it is and reports it in `rewrote`.
That is a third option the rule does not offer, and no `Pattern-Departure:` trailer names it.
A value of 0 is different: it meant "off" before and after, so dropping it carries no meaning across.
It is the change the board entry asks for ("the fix sets it back to a working threshold"), and the hub file at `_bolt/_lym/config/shuttle.yaml:1` holds exactly that 0.
The drop of 0, and of a null value, which loads as 0, stays.
Only a non-zero count needs the flag.
Creel's reinterpretation also drops entries, but only ones that name no repository, so nothing the hub meant is lost there.
A dropped `edit_loop_edits: 6` loses the hub's choice.

### [MEDIUM] A repeat expiring out of the window has no test

**Location:** `internal/shuttleengine/editloop.go:471`

**Issue:** `fold` prunes both `history.contents` and `history.repeats` against the current edit's time.
The row "content older than the window before the latest edit drops out of the count" pins the contents pruning.
No row pins the repeats pruning.
The stamped 0/120/240/360 row never forms a repeat.
The step-5/12 row forms one repeat, which stays below the count of 2 either way.
Without the pruning, two short back-and-forths more than a window apart would add up to one false notice: the noise this task exists to remove.

### [LOW] The hook-edit help text describes the old count

**Location:** `internal/shuttlecli/hookedit.go:26`

**Issue:** The verb's `Long` text says it "appends one record of the edited file to the run's edit records file" and that the wait counts those records "to tell its parent about an agent that edits one file again and again".
After this diff the record carries a content hash, and the watch counts only edits that return a file to earlier content.
`docs/overview.md` and both `doc.go` files were updated, but this help text was not.

### [LOW] The EditRecorder seam contract leaves out the content hash

**Location:** `internal/shuttleengine/engine.go:314`

**Issue:** The interface doc describes the hook as recording "each file edit", read back as a `SessionSignalFileEdit`.
`fold` now skips every file-edit signal whose `ContentHash` is empty (`editloop.go:458`).
An engine that implements the seam from this contract alone would get a watch that never fires.

### [LOW] TestDropEditLoopEdits has no row for a null value or a non-mapping document

**Location:** `internal/shuttleengine/reinterpret_test.go:791`

**Issue:** The focus asks how a null or missing key and a malformed document are handled.
A null `edit_loop_edits` decodes to a non-zero `yaml.Node` and is dropped, and a top-level sequence fails `yaml.Unmarshal` into the struct and is marked invalid.
No row pins either path.

### [NIT] The orch line names an empty value for a non-scalar or empty edit_loop_edits

**Location:** `internal/shuttleengine/reinterpret.go:753`

**Issue:** `values.EditLoopEdits.Value` is empty for `edit_loop_edits:` with no value and for a mapping or sequence node, so the mailed line reads `edit_loop_edits  dropped`.
Every such value is a config error at load, so the effect is cosmetic.

### [NIT] A carried-text check that can never fail

**Location:** `internal/shuttleengine/claudeengine/editrecord_test.go:109`

**Issue:** No payload or file `TestRecordEdit` writes contains `package b`, so that entry of the carried list asserts nothing.

## Verdicts on the focus directives

- **Repeat count against the board entry.**
  An A→B→A→B→A flip counts repeats 0, 0, 1, 2, 3 and fires on the fifth edit at the template's 3, matching the summary.
  An identical `Write` counts as a repeat.
  The undo of the first edit is invisible, since pre-edit content is never hashed; the summary records this, and a loop that keeps going is caught from its third edit.
  An edit that repeats the same `old_string` but leaves new content each time, such as one insertion applied twice, is not counted.
  The board entry's examples are both returns to earlier content, and it offers them as alternatives, so this is not a defect.
- **Window pruning, reset and clone.**
  A signal with no `At` takes `now`, and an empty `ContentHash` is skipped before any state changes.
  Keys separate agent and file; the "count apart" rows use equal hashes, so a merged key would fail them.
  `clone` copies both slices, and `before` is taken before `fold` mutates the shared arrays in place, so the restore on a failed save is sound.
  The reset deletes the whole history; the "twice as long" row pins it.
  The only gap is F2.
- **Hashing at hook time.**
  The edit hook is a plain synchronous `PostToolUse` command with no `async` or `asyncRewake` flag (`claudeengine/settings.go:511`), so it reads the file before the next serial `Edit` or `Write` runs.
  Claude Code's `Edit` and `Write` take an absolute `file_path`, so a relative path does not arise.
  A large file is streamed through `io.Copy`.
  An unreadable or removed file yields no hash, and that edit never counts.
  The hash reaches `SessionSignal.ContentHash` through `fileEditSignal`, which `TestRecordEdit`'s readable row pins end to end.
  One limit is not a defect of this diff: two agents editing one file at the same instant could each hash the other's write.
- **Reinterpretation once, and the hub default.**
  `hubreconcile.applyReinterpretations` writes the board marker after the rewrite, so a later walk skips it.
  The hub's `edit_loop_edits: 0` is dropped and the template's 3 applies, as the board entry asks.
  The non-zero path is F1.
  The null and malformed paths behave correctly but are untested (F5).
- **Scope.**
  Every change traces to the board entry's Idea or to the threshold reset, or is forced by one of them.
  The fallback of exempting roles is rightly not built.
  `docs/overview.md`, both `doc.go` files, `Config.EditLoopEdits`, the template comment and the notice wording carry the new semantics.
  The hook-edit help text and the seam doc do not (F3, F4).
- **Test economy.**
  The progress-making fixer case is the row "edits that each leave new content make progress and do not notify".
  The revert case is the `flip` rows.
  Every test the redundancy report lists is unique, and none adds an assertion on static prose.
  The missing rows are F2 and F5.

## Rejected

- The code-scope lens reported no findings, and the other lenses reported nothing that is rejected here.
