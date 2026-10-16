---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-claude-restored-prompt-clear.md:24
    summary: RestoredPromptClearSequence counts only interior rows with text, so a restored prompt with a blank row between paragraphs gets fewer C-u/BSpace pairs than rows and is left partly uncleared, against D3's one C-u per line of the box's text
    fix: In card 4's Intent, size the sequence from every interior row between the first and the last row that holds text once the caret marker, side bars and whitespace are stripped, blank rows between them included (n = last minus first plus one, no keys for a box with no such row); keep the stated bound of at most two keys per box line the capture shows. Add a row to card 4's RestoredPromptClearSequence table in TestClearInputSequence for a three-row box whose middle row is blank, answering C-u BSpace C-u BSpace C-u. In card 5, make restoreEngine's RestoredPromptClearSequence follow the same count, and add a TestRunner_InterruptRestoring row where a restored prompt with a blank middle row is emptied.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/02-claude-placeholder-blank.md:31
    summary: StyledBoxReader promises the plain capture, but tmux capture-pane -e keeps trailing blank cells that carry a non-default background, which capture-pane -p trims, so the escape-stripped styled capture is not the plain capture every D1 box reading used before
    fix: In card 2's Intent, make Claude.PlainBoxCapture trim trailing whitespace from each line after the escapes are removed, as capture-pane -p does, and state that in the StyledBoxReader doc. Add a row to TestPlainBoxCapture whose styled line ends in spaces drawn with a background SGR (for example ESC[44m followed by text and three spaces, then ESC[49m) and whose answer line carries no trailing spaces.
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/plan/12-daemon-log-file.md:54
    summary: D5 says a give-up error says the log could not be opened when the spawn could not open it, but card 12 says so only when the client cannot read the file, so an existing log the spawn could not append to or truncate reads as holding no line from that start
    fix: In card 12's Intent, make daemonLogExcerpt first open the told log path for append, as openDaemonLog does, and say the daemon log could not be opened when that open fails, before it reads the file; keep the unreadable-file wording. Add a row to the TestDaemon_Lifecycle give-up table where the log file exists, holds only pid 77's lines and has mode 0o444, and the error says the daemon log could not be opened.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/05-shuttle-interrupt-restoring.md:48
    summary: Runner.InterruptRestoring prescribes refusing an untracked guid with notShuttleOrLiveStrandError, but no TestRunner_InterruptRestoring row reaches that refusal
    fix: Add a row to card 5's TestRunner_InterruptRestoring list where the guid has no run record and the reed reports no live strand for it (a newFixture without withStrand whose fakeReed StatusQueue lists no such strand), asserting the call fails with notShuttleOrLiveStrandError's error naming the guid and sends no key.
---

### [MEDIUM] The restored-prompt clear undercounts a prompt with a blank row

**Location:** `_lyx/plan/04-claude-restored-prompt-clear.md:24`

**Issue:** Card 4 counts a box line "once the caret marker, side bars and whitespace are stripped and something remains".
`inputBoxInterior` (`internal/shuttleengine/claudeengine/session.go:210`) returns every row between the top and bottom rules, so a multi-paragraph prompt restored after the interrupt has blank interior rows.
Card 4 does not count those rows, so the sequence has fewer `C-u`/`BSpace` pairs than the box has rows.
With the live-confirmed key behaviour (one `C-u` deletes the caret row, a `BSpace` at a row start joins the row above), each row costs one pair, blank or not, so the top rows stay.
D3 sizes the clear as "one `C-u` per line of the box's text", and D3's bound already allows two keys per line the read showed.
The failure is safe, since card 5 sends nothing more and D2 reports a draft, but it misses D3's goal for an ordinary multi-paragraph prompt.
Neither card 4's nor card 5's rows include a blank row, so no test would catch this.

### [LOW] An escape-stripped styled capture is not the plain capture

**Location:** `_lyx/plan/02-claude-placeholder-blank.md:31`

**Issue:** Card 2's `StyledBoxReader` "answers the plain capture, every SGR and other escape sequence removed", and card 3 then feeds that answer to every Claude reading on the D1 paths in place of `capture-pane -p`.
I ran a private tmux server (`tmux -L burlerprobe`) that printed a line ending in three spaces under `ESC[44m`.
`capture-pane -p` returned `blue bg`, and `capture-pane -e -p` returned `ESC[44mblue bg   ESC[49m`, keeping the trailing cells.
Stripping the escapes therefore leaves trailing spaces the plain capture never had wherever Claude draws a background colour, such as diff rows in the transcript.
Card 2's own row compares the answer with "the fixture with its escapes removed", so the plan never checks that the answer equals what `-p` returns.

### [LOW] The could-not-open wording is decided by the reader, not the spawn

**Location:** `_lyx/plan/12-daemon-log-file.md:54`

**Issue:** D5 says that when the log cannot be opened, "the daemon starts with null stderr, and a give-up error says the log could not be opened."
Card 12 says that only for "a file that cannot be read", on the assumption that a spawn that could not open the log leaves an unreadable file behind.
An existing file that the spawner cannot open for append, or cannot truncate past the cap, is still readable.
In that case the excerpt answers that the file holds no line from that start, which hides the real cause D5 says to name.

### [LOW] No row reaches InterruptRestoring's untracked-guid refusal

**Location:** `_lyx/plan/05-shuttle-interrupt-restoring.md:48`

**Issue:** The new entry point refuses a guid that has neither a run record nor a live strand, through `notShuttleOrLiveStrandError`.
Card 5's row list (lines 72-85) covers a strand with no run record that is live, but no row reaches the refusal.
The rubric requires each prescribed refusal of a new entry point to name a test that reaches it.

## Rejected

- Card order: no card uses a symbol that a later card creates or changes.
  Card 3 uses card 2's `StyledBoxReader`, card 4 uses card 2's `PlainBoxCapture`, card 5 uses cards 3 and 4, cards 7 and 8 use card 6's seam, card 8 uses card 5's entry point, and card 13 uses card 7's `runCreelSplit` and `lockedBuffer` and card 12's line format.
- Card 1's `ReedOps` implementer list is complete: only `reedengine.Engine`, `fakeReed` and `shuttlefake.Reed` define `SetWaitMark`, and every other test reed embeds `*fakeReed` or `*tuiReed`.
- Card 12's `NewClient` caller list is complete; the `nil` spawn in `TestDaemon_Protocol` at `server_test.go:403` still compiles against the new function type.
- Verify: there is no nested module (`tools/sandbox` has no `go.mod`), `go test ./...` and `-tags integration` run every targeted package, `llm` and `tmux` are vetted, and the order matches the template.
- Card 7's progress lines reach real stderr in production, because `cmd/lyx/main.go:55-56` sets stdout and stderr separately.
- Card 6's too-short way forward, `lyx reed attach`, is not in the deny set, and reed pins `window-size latest`, so attaching from a taller terminal grows the pane.
- Card 8's per-receiver lock adds no lock-order inversion: `interruptTarget` runs outside the store mutex, and both sides take the pane lock before `Store.do`.
- Test budget: the rows cited in cards 1, 2 and 5 (269, 113, 178) match `cmd/lyx/testdata/test-budget.yaml` and the current top-level test counts.
- Fidelity: D1 to D9 and every Constraint are carried, and no card changes this task's board entry.
  Card 3's `internal/orchengine/doc.go` edit is a doc consequence of the pane-only `Draft` change, not new scope.
