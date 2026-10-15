---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md Acceptance criteria, creel interrupt bullet (multi-line restored prompt)
    summary: The multi-line acceptance criterion runs on the shuttle fake, whose C-u empties the whole box, so a one-C-u clear passes it and a multi-line prompt left behind on Claude fails no criterion
    fix: In decision-record.md Acceptance criteria, replace "a multi-line restored prompt is emptied by creel's interrupt clear on the shuttle fake" with a criterion that pins the sequence and the fake behaviour, for example "for a box read as n lines, the restored-prompt clear sends n C-u with one BSpace between each pair and nothing else; on a fake whose C-u deletes only the caret's line and whose BSpace at a line start joins the line above, a two-line and a three-line restored prompt are emptied, and a single C-u leaves text". Keep D3's own sequence text as it is.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 and its Acceptance criterion (daemon log names the file, never a line from another start)
    summary: Two clients spawning at once append two daemons' headers and lines to one file interleaved, so quoting the lines after this start's header can quote the other start's lines, which the criterion forbids
    fix: In decision-record.md D5, decide that every line the daemon writes through its logger carries its pid, and that the client quotes only lines carrying the pid it spawned plus pid-less lines (a runtime panic trace) that follow its own header with no other start header after it; when another header follows, the error quotes the pid-tagged lines and says the file also holds unattributed lines. Extend D5's Bound to say that a truncation erasing this start's header but not its later lines still finds them by pid. Keep the Acceptance criterion's never-a-line-from-another-start wording, which then holds.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3, the sentence beginning "A restore that lands between the guard's read and the paste-submit"
    summary: D3 says the restore-during-paste window is bounded by the two settles, but the settles only set the earliest time the window opens; the window itself is the send's guard-read-to-paste interval
    fix: In decision-record.md D3, rewrite the sentence as "A restore that lands between the pre-write guard's read and the paste-submit is submitted with the paste; that window opens no earlier than two settles after the key and lasts only from the guard's read to the paste, and its worst outcome is the stopped turn's own prompt re-sent beside the inject, never an operator's draft."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 parenthetical list of box reads, and D3's before-key, post-settle and post-clear reads
    summary: D1 lists the box reads that take the blanked view and D3's three box reads are not among them, so on a hand-added strand a placeholder or prompt suggestion read plain counts as text, contradicting D3's bound that the cleared text is the stopped turn's prompt
    fix: In decision-record.md D1, add "creel's interrupt reads before the key, after the settle and after the clear" to the parenthetical list of box reads that derive the plain view from one styled capture.
---

### [MEDIUM] The multi-line clear criterion passes with a single C-u

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, creel interrupt bullet.

**Issue:** D3 now decides the multi-line clear itself: the sequence (one `C-u` per line, one `BSpace` between each pair), its bound (deletes only, no Escape, `C-c` or Enter, at most two keys per line) and its fallback (nothing more sent, D2 shows a draft).
The criterion meant to pin it is "a multi-line restored prompt is emptied by creel's interrupt clear on the shuttle fake".
The shuttle fake's `tuiReed.SendKey` (`internal/shuttleengine/fakes_test.go`) sets `r.box = ""` on any `C-u`, so a clear of one `C-u`, today's `ClearInputSequence`, empties a multi-line box on the fake.
The criterion therefore cannot fail if the implementation reuses the one-key clear and leaves a multi-line prompt behind on Claude, which is the defect the focus asks it to catch.
The live check exists only as an Auto-mode assumption the plan confirms, not as a criterion.

### [MEDIUM] D5's start header does not separate two concurrent starts

**Location:** `_lyx/discussion/decision-record.md` D5 and the daemon-log Acceptance criterion.

**Issue:** `spawnDaemon` (`internal/creelcli/spawn.go`) starts the binary directly, so `cmd.Process.Pid` is the daemon's pid and the header's pid is a sound key for one start.
Under two clients spawning at once, both daemons append to the one file: a losing daemon exits with `ErrDaemonRunning` (`internal/creelengine/doc.go`), and both write a header and their start lines.
Those lines interleave, and D5 quotes "the lines after the start header carrying the pid", with no end marker and no pid on the later lines, so client A can quote daemon B's header and error.
That breaks the criterion "quoting its error line from that start, never a line from another start".
On truncation, D5's Bound holds when the other client's truncation erases this start's header, since the error then says the file holds no line from that start.
It is misleading when the truncation lands between this start's header and its error line: the error line survives but cannot be attributed, and the client says there is none.
Pid-tagged lines fix both cases. Only a runtime panic trace carries no pid, which is why the fix keeps a rule for pid-less lines.

### [LOW] D3 misstates what bounds the restore-during-paste window

**Location:** `_lyx/discussion/decision-record.md` D3.

**Issue:** "that window is bounded by the two settles D3 waits before waking the worker".
The two settles decide only how late after the key a restore must arrive to reach this window.
The window's length is the resumed send's interval from the pre-write guard's read to the paste-submit, which D3 does not change.
A restore earlier than that read is seen as a draft and waits, as D3 says.
The stated worst outcome is right, and the timing assumption is stated under Auto-mode assumptions ("within its `InterruptSettle`").

### [LOW] D3's box reads are not among D1's blanked reads

**Location:** `_lyx/discussion/decision-record.md` D1 and D3.

**Issue:** D1's parenthetical names the idle probe, the pre-write guard and the landing check, and says every other classifier keeps the plain capture.
D3's reads before the key, after the settle and after the clear are box readings too, and the acceptance criterion "The creel interrupt clears a restored prompt on a strand with no run record" targets hand-added strands, which run with prompt suggestions on (#657).
Read plain, a placeholder or suggestion counts as text: before the key it suppresses the clear, and after the settle it fires a clear on text that is not the stopped turn's prompt, contrary to D3's bound.
D1's "Wherever shuttle reads the input box" may already mean these reads, but the list beside it reads as closed.

## Focus verdicts

- D3, multi-line clear: decided in D3 itself, with the sequence named, the bound stated (it deletes box text only, sends no Escape, `C-c` or Enter, and sends at most two keys per line) and a fallback. The criterion does not fail on a left-behind multi-line prompt (F1).
- D3 and Auto-mode assumptions, redraw timing: the assumption is stated, and D3 says what happens to a later redraw (one more settle and a read, then the pre-write guard) and states the worst outcome. The window's bound is misworded (F3).
- D5, start header under concurrent clients: the pid is a sound key for one start, but the lines after the header are not attributed under interleaving (F2). The Bound line is true when the header is truncated away, and misleading when only the header goes (F2).
