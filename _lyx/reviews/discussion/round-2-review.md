---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (Open risks, first bullet)
    summary: The C13 risk hedges that the stacked 3-row strand may no longer be reachable, where the other task's plan removes every way to reach it, and the follow-up it calls for has no owner.
    fix: In decision-record.md's Open risks, replace the sentence starting "Its D4 removes the shell window's stacked strand layout" with a certain statement — once reed-add-parentless-placement lands, C13's stacked 3-row half cannot be set up, since its cards 2 and 4 remove the stack and `collapsed_rows`, card 5 refuses `below-parent` and `top`, and card 7 moves any strand pane left in the shell window into its own window; C13's hand-added half stays reachable. Then add a Decision (D6) that this task files one board note through `lyx board` for rewriting C13's stacked half against the window-per-strand layout, and have the Open risks bullet point to that note instead of "a follow-up outside both tasks".
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D1, the No controlled tmux exceptions bullet)
    summary: D1 drops the tmux exception because no creel scenario lists panes, but S9, C2, C6 and C7 judge what a pane shows, and the standalone suite's black-box rule leaves the agent no lyx verb to read a pane.
    fix: In D1's No controlled tmux exceptions bullet, keep the omission and add one sentence saying that pane content the creel scenarios judge (S9's typed reminder, C2's single reminder in the scrollback, C6's line not typed again, C7's restored prompt) is observed by the operator through `lyx reed attach` and reported back by the agent, as C12's operator-assisted check is, so the creel suite needs no tmux read exception. Write the same rule into the creel suite's Controlled creel exceptions section as part of D1's list of what that section holds.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Open risks, second bullet)
    summary: The daemon bullet names only the open lock and log file, but the daemon also holds the current day's journal file open in the same state directory.
    fix: In decision-record.md's Open risks second bullet, replace "its open lock and log file in the hub's creel state directory" with "its open `daemon.lock`, `daemon.log` and current journal day file under the hub's creel state directory, `_board/.lyx/creel`".
---

### [MEDIUM] C13's stacked half is certainly unreachable after the other task lands, and its follow-up has no owner

**Location:** `_lyx/discussion/decision-record.md`, Open risks, first bullet.

**Issue:** The bullet's first claim holds: card 8 of `reed-add-parentless-placement` lists only `SANDBOX-REED-SUITE.md` and `SANDBOX-REED-WATCH-SUITE.md` under Prosa, no other card names a shuttle or creel suite file, and its verify grep (overview, line 39) only checks for `below-parent`, `anchor: top` and `--anchor top`, which none of the moved lines contains.
The second claim understates the case.
C13's Watch needs "a stacked 3-row strand", and that plan removes every way to make one:
card 2 deletes render's stack, height rules and `planLayout`;
card 4 retires `collapsed_rows`;
card 5 refuses `below-parent` and `top` on write and in `lyx reed add`;
card 7 breaks any bound strand pane left in the shell window out into its own window on every reconcile.
So "may no longer be reachable" is a false hedge: after that task lands, C13's stacked half cannot be set up, while its plain `lyx reed add --cmd claude` half stays reachable.
"C13 then needs a follow-up outside both tasks" names no owner and no board entry, and Plan-Write never reads the support log, so nothing carries it once this run lands.
Filing a separate board note is a write to another entry, which the run's board lock allows.

### [LOW] D1 drops the tmux exception without saying who reads pane content

**Location:** `_lyx/discussion/decision-record.md`, D1, the No controlled tmux exceptions bullet.

**Issue:** D1's reason, "no S9 or C scenario lists panes", is true for listing.
But S9 ("a one-line reminder ... appears typed in its pane"), C2 ("appears once in the pane's scrollback"), C6 ("the line is not typed again once the box is cleared") and C7 ("Claude puts the interrupted prompt back in the input box") judge a pane's content.
`lyx reed` has no capture verb (its verbs are attach, add, remove, resume, follow, list, switch, up, down, status, watchdog), and neither has `lyx creel`.
In the combined suite, S5 and S6 put `tmux -L <socket> capture-pane` in their own Watch text beside the tmux exception, so an agent there saw pane reading in use.
The standalone creel suite carries neither, so its black-box rule leaves the agent with no way to make these checks, and D1 should say how they are made.
Making them operator-observed adds no exception and matches the suite's "an agent drives it, an operator watches" model, so it needs no attack-surface bound.

### [LOW] The daemon bullet omits the open journal file

**Location:** `_lyx/discussion/decision-record.md`, Open risks, second bullet.

**Issue:** `internal/hubgeom/creelgeom.go` puts `daemon.lock`, `daemon.log` and `journal/` under the state directory `<hub>/_board/.lyx/creel`, as `internal/creelengine/geometry.go` documents.
Besides the lock `Run` holds and the stderr log, `journalWriter.rotate` in `internal/creelengine/journal.go` opens the current day file and keeps it in `w.file` until the next rotate or `close`, so the daemon holds three files open there.
The bullet's "stops only when its binary is replaced" matches `doc.go`.

## Rejected

- D1's carried creel exceptions (focus 2): each is used by a scenario, the socket-directory listing by C1, the day-file planting and editing by C10 and the detached process and its operator-assisted fallback by C12. None of their bounds depends on a shuttle strand guid, so each applies to a creel run as written. This is not a defect.
- Card 8's sweep editing a moved line (focus 1): it does not, as F1's first paragraph shows.
