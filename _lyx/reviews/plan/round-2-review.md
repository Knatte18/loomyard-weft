---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/plan/02-claude-placeholder-blank.md:34
    summary: parseStyledLine reads SGR parameter 2 as faint with no rule skipping extended-colour sub-parameters, so text drawn in an RGB or 256-colour style can read as faint and be blanked
    fix: In card 2's paragraph on parseStyledLine, after "the other parameters leave it" add that the parameters after 38, 48 and 58 are consumed as a colour and never read as attributes, that is "38;2;r;g;b" and "38;5;n" (and the same for 48 and 58), so a 2 or a 0 inside a colour sets and clears nothing. In the same card's TestPlainBoxCapture row list add one row whose box text is drawn under "ESC[38;2;10;2;20m" and then under "ESC[38;5;2m" with no faint, expecting the answer unblanked, plus a row where faint text is followed by text under "ESC[38;5;0m", expecting that text to stay faint and the box to blank, so the skip is pinned in both directions. No target changes, so no validate-plan overlay is needed.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-daemon-log-file.md:55
    summary: the prescribed branch where the log opens for append but cannot be read has no named input or test that reaches it
    fix: In card 12's Tests paragraph, add a row to the table over ClientWithNoListenerSpawnsOnceAndGivesUpNamingTheSocket whose log file has mode 0o200 (writable, not readable) and expects the error to say the daemon log could not be opened and to name the log file. Say the row skips on Windows and when os.Geteuid() is 0, because Windows has no write-only file and root reads any file. If the plan prefers not to carry that row, drop the "cannot be read" sentence from the daemonLogExcerpt paragraph instead and let a read failure fall under the "holds no line from that start" answer. The row is an addition to a target the card already lists, so every plan check admits it.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/12-daemon-log-file.md:68
    summary: the 0o444 row says nothing about root or Windows, where the append-open it expects to fail succeeds or behaves differently
    fix: In card 12's row list, after "the same file with mode 0o444 says the daemon log could not be opened;" add that the row skips on Windows and when os.Geteuid() is 0, as the repository's other permission-denial rows do (for example the guard in internal/shuttleengine/rundir_test.go and internal/loomcli/fmtverb_test.go). Root opens a 0o444 file for append, so the row would otherwise read "holds no line from that start" and fail.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/02-claude-placeholder-blank.md:35
    summary: the trailing trim is worded as whitespace, but capture-pane -p trims only ASCII spaces and keeps the NBSP Claude draws after the caret, so the StyledBoxReader equality claim is not exact
    fix: In card 2, change "trimming the trailing whitespace" (line 35) and the StyledBoxReader doc wording "each line's trailing whitespace trimmed as capture-pane -p trims it" (line 31) to "trailing ASCII spaces (U+0020) trimmed", and add one TestPlainBoxCapture row whose box line is the caret, an NBSP and nothing else under no SGR, expecting the answer to keep the NBSP exactly as capture-pane -p returns it. Confirmed against tmux 3.6, where the plain capture of a line ending in an NBSP keeps it and the plain capture of a line ending in blue-background spaces drops them.
---

### [MEDIUM] SGR faint tracking can read an RGB or 256-colour style as faint

**Location:** `_lyx/plan/02-claude-placeholder-blank.md:34`
**Issue:** The card tracks faint as "parameter `2` sets it; `0`, `22` and an empty `ESC [ m` clear it; the other parameters leave it".
Read as a flat parameter list, that mis-reads colour sequences: tmux emits an RGB colour as `38;2;r;g;b` (the card's own fixture has `ESC[38;2;153;153;153m` on the status row) and a 256-colour as `38;5;n`.
The `2` of `38;2` would set faint, and a `0` in `38;2;0;…` or `38;5;0` would clear it.
Box text drawn in an extended colour, such as Claude's highlighted tokens, then reads as faint, and a box holding only such text is blanked and read as empty.
That is the fail-open direction D1's bound forbids: "Text in any other style … never blanked".
No row in `TestPlainBoxCapture`, `TestIdleSession` or `TestInputBoxText` draws an extended colour in the box, so an implementation with the flat reading passes every listed row.

### [MEDIUM] The "opens for append but cannot be read" branch of the give-up excerpt is unreached

**Location:** `_lyx/plan/12-daemon-log-file.md:55`
**Issue:** The card prescribes that a file that opens for append but cannot be read says the daemon log could not be opened.
The row list names a readable but unwritable file (0o444), an absent file and readable files with or without matching lines.
No named input makes the append-open succeed and the read fail, so the branch has no test that reaches it.
It is not a propagated raw I/O error of a section in `refusal-spec.md`, so the exemption does not apply.

### [LOW] The 0o444 row needs the repository's root and Windows skip

**Location:** `_lyx/plan/12-daemon-log-file.md:68`
**Issue:** The row's premise is that opening the file for append fails.
Root opens a 0o444 file for append without error, so the row fails under a root gate environment, and the repository's other permission-denial tests skip for `os.Geteuid() == 0` (and Windows).
This row is the one that settles the focus question of an existing readable file the spawn could not append to, so it must stay deterministic.

### [LOW] Trailing-whitespace trim is not what `capture-pane -p` does

**Location:** `_lyx/plan/02-claude-placeholder-blank.md:35`
**Issue:** The equality the card and the `StyledBoxReader` doc claim holds for lines ending in background-coloured blank cells: I ran `capture-pane -p` and `capture-pane -e -p` on tmux 3.6 over a pane with `ESC[44m` text and trailing spaces under `ESC[49m` and `ESC[0m`, and the plain capture was the escape-stripped styled capture right-trimmed of spaces.
It does not hold for a line ending in an NBSP: the plain capture keeps the NBSP, and Go's whitespace trimming would drop it.
Claude draws an NBSP after the caret (the card's own live-capture fact), so an empty unsuggested box row is `❯` plus NBSP.
The readings downstream strip whitespace with `strings.TrimSpace`, so behaviour does not change, but the stated contract does, and no row pins it.

## Rejected

- Focus: `RestoredPromptClearSequence` sizing, cards 4 and 5. Not a defect. Card 4 counts rows from the first to the last interior row where something remains after the caret, side bars and whitespace are stripped, blank rows between them included, and returns n `C-u` with one `BSpace` between each pair.
  Card 4's table has a three-row box with a blank middle row answering `C-u BSpace C-u BSpace C-u`, and card 5's `TestRunner_InterruptRestoring` has a three-row restored prompt with a blank middle row on a fake whose `BSpace` at an empty last row removes that row, so both cards reach the multi-paragraph case.
- Focus: escape-stripped styled capture against `capture-pane -p`, cards 2 and 3. Not a defect for background-coloured blank cells: verified live on tmux 3.6, and the card's `ESC[44m` row pins it. The NBSP gap is F4.
- Focus: give-up error when the spawn could not open the log, card 12. Not a defect for an existing file the spawn could not append to: `daemonLogExcerpt` opens the path for append without creating it, as `openDaemonLog` does, and the 0o444 row reaches it (determinism of that row is F3, the unreadable sub-branch is F2).
- Focus: refusal of a guid with no run record and no live strand, card 5. Not a defect: the last row of `TestRunner_InterruptRestoring` uses a `newFixture` without `withStrand` and `liveStrandStatus(false)`, `FindRun` returns `ErrNoRunRecord` for it, `liveStrand` fails, and the row reaches `notShuttleOrLiveStrandError` with no upstream check rejecting it.
