---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md D1 and Acceptance criteria (first bullet)
    summary: D1 prescribes the wording never an inline `python` or `node` heredoc, which puts two words between never and the closed-list form `python`, so the PATTERN-no-denied-recovery scan fails the directive D1 claims it admits
    fix: In D1, replace the prescribed wording never an inline `python` or `node` heredoc with never `python` or `node` in an inline heredoc (never directly before `python`, as the current directive has it), and in the first acceptance criterion replace and the interpreter heredoc with and `python`, so the criterion names the closed-list form the scan matches rather than a phrase it does not.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6 (latch bullets)
    summary: The latch clears only on a pane read that shows an input box, but pane reads stop once the notice is sent, so after the first notice the latch never clears and no later dialog in the same run is ever notified
    fix: "In D6, add to the third latch bullet: after the notice is sent, the loop keeps reading the pane on each poll tick until a read shows an input box, which clears the latch so a later ask starts a new episode. Add an acceptance criterion: a run whose first dialog was notified and then answered (pane shows an input box), followed by a second ask, sends a second notice once the bound passes on that second ask."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D6 (persisted offset and restart rebuild)
    summary: One dialog yields several ask events (a PermissionRequest and a permission_prompt Notification both fold to ask), and the persisted offset is the latch's first ask, so a restart rebuilds a latch from the later ask of an already-notified dialog and repeats the notice, contradicting the no-repeat acceptance criterion
    fix: "In D6, state that when the notice is sent the persisted offset is the offset of the latest ask folded into the latch episode up to that moment, not the episode's first ask, and that on attach an ask at or before that offset is not rebuilt into a latch. Extend the reattach acceptance criterion: a run reattached after the notice was sent, whose events hold a PermissionRequest and then a permission_prompt Notification for the same dialog, does not repeat the notice."
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D2 (pattern and Bound) and Acceptance criteria (in-place deny bullet)
    summary: D2 matches an in-place flag anywhere among the sed words without saying where those words end in the raw payload, and its Bound omits let-throughs and false denies that follow from the described pattern
    fix: "In D2, state that the sed and perl words run to the next `;`, `&`, `|`, `)`, backtick or JSON-escaped newline, and that a flag word must open after whitespace with a single dash (so `--quiet`, `--silent`, `--expression=` and `--file=`, which hold an i, never match). Extend the Bound's lets-through list with `find -exec sed -i` and a prefix the command-position pattern does not list (`nohup`, `command`, `timeout`), and its false-deny list with a perl argument cluster holding i such as `-Mstrict`. Add to the in-place deny acceptance bullet that it admits `sed --quiet 3p f`, `sed --silent 3p f` and `sed -n 3p f` followed by a newline and `grep -i x f`."
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D7
    summary: D7 adds only the bare spellings `sed -i` and `perl -i` to the whole-token closed list, so a stencil offering `perl -pi -e`, `sed -ni` or `sed --in-place` as a step passes the scan although D2 denies it, and D7 states no bound for that gap
    fix: "In D7, state that the closed-list entries `sed -i` and `perl -i` catch only those bare spellings and that the other spellings D2 denies (`-i<suffix>`, `--in-place`, a short cluster holding i such as `-ni` or `-pi`) stay outside the token scan, as the raw `go` deny does, and require the background file to say so in the same sentence it adds for the computed-path `rm` deny."
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3 (Bound)
    summary: D3's lets-through list omits positional and special parameters such as `rm $1`, `rm $@` and `rm $*`, which are computed paths the described pattern, `$` followed by a letter, `_` or `{`, does not match
    fix: In D3's Bound, add to the lets-through list a removal through a positional or special parameter (`$1`, `$@`, `$*`), or widen the pattern to `$` followed by a letter, digit, `_`, `{`, `@` or `*` and add `rm $1` to the computed-path deny acceptance bullet; pick one and say which.
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md Goal, D3 Rationale and Auto-mode assumptions
    summary: The Goal says the shell forms that raise a prompt are steered away before they run, but D3 covers only rm with a substitution or variable, and no assumption names which forms Claude Code's statically-unresolvable check fires on
    fix: Add an Auto-mode assumption that Claude Code's statically-unresolvable rm prompt fires only on a target holding a command substitution or a shell variable, as D3's pattern assumes, and that another form (a glob, `~`, a positional parameter) left to it still stalls the run until D6's notice; in the Goal, change the shell forms that raise one to the `sed -i` and computed-path `rm` forms that raise one.
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D1 (second paragraph)
    summary: D1 does not say whether the `.scratch/` exclude entry is anchored at the worktree root or matches at any depth, nor that files under it escape git-status dirty-tree checks, so the plan writer must choose the edge's reach unbounded
    fix: In D1's second paragraph, state whether the entry is root-anchored (`/.scratch/`) or bare (`.scratch/`), and add one sentence of bound saying what it lets through, untracked files under `.scratch/` that a dirty-tree check no longer sees, and that it never hides tracked content, since git ignores no tracked path.
---

### [BLOCKING] D1's interpreter wording fails the no-denied-recovery scan

**Location:** `_lyx/discussion/decision-record.md` D1 and the first acceptance criterion.

**Issue:** D1 says each rewriting form sits directly after its own "never" so the exemption marker admits it, then prescribes "never an inline `python` or `node` heredoc".
`python` is on the closed list in `cmd/lyx/norecoverydeny_test.go`, and `deniedRecoveryMarker` exempts a form only when "never " precedes it directly, with at most one backtick between.
"an inline " sits between them, so the scan reports a hit in `contracts/stencils/edit/edit-directive.md`, which is in the scanned surface.
The current directive reads "never `python` or `node` in an inline heredoc", which passes; D1's rewording breaks it.
The acceptance criterion repeats the problem by requiring "never" before "the interpreter heredoc", which is not a form the scan knows.
A plan writer copying D1 verbatim lands a `PATTERN-no-denied-recovery` violation.

### [MEDIUM] D6's latch never clears after its first notice

**Location:** `_lyx/discussion/decision-record.md` D6, latch bullets.

**Issue:** The second bullet says the latch clears only when a pane read shows an input box.
The third says the loop reads the pane "until it shows an input box or the notice is sent".
Once the notice is sent, reading stops, so nothing can clear the latch, and the first bullet makes every later ask a no-op while it is set.
In a Webster run with many forks, a second dialog after an answered first one is never notified, which is the stall the Goal says must reach the operator.
The Bound's "one line per latch episode" assumes episodes end; as written, the first notified episode lasts the rest of the run.

### [MEDIUM] D6's restart rebuild repeats a sent notice

**Location:** `_lyx/discussion/decision-record.md` D6, persisted offset and attach rebuild.

**Issue:** `claudeengine`'s `doc.go` folds a `PermissionRequest` and a `permission_prompt` `Notification` each to an ask, so one dialog commonly yields two ask events.
D6 stamps the latch with the first ask's offset, ignores the later ask while set, and persists that first offset.
On attach it rebuilds from events after the persisted offset, finds the second ask of the same, still-open dialog, and stamps a new latch with that ask's old time.
The bound has passed and the pane shows no input box, so the run sends a second notice.
That contradicts the acceptance criterion that a run reattached after the notice was sent does not repeat it.

### [MEDIUM] D2 leaves the extent of the sed words open, and its Bound is incomplete

**Location:** `_lyx/discussion/decision-record.md` D2 and the in-place deny acceptance bullet.

**Issue:** D2 matches an in-place flag "anywhere among its words" with a grep over the raw payload JSON, but never says where the sed command's words end.
`sed -n 3p f -i` must be denied, so the match runs past the script and file words; with no stated stop, a later command's `-i` after a JSON-escaped newline or inside a subshell is a false deny the Bound does not list.
"A short-flag cluster holding `i`" is safe only if it is anchored to a single-dash word after whitespace.
`--quiet` and `--silent` hold an `i`, and D5 promotes both as read forms, yet no acceptance criterion pins that the deny admits them.
The Bound's lets-through list stops at redirection, `w`, `awk`, `bash -c`, `eval` and a variable.
The `commandPositionPattern` it reuses also lets through `find -exec sed -i`, the common bulk-edit spelling, and a prefix it does not list, as the python row's comment in `settings.go` records for that row.
For `perl`, "i in a short-flag cluster" also catches a read such as `perl -Mstrict -ne ...`.

### [MEDIUM] D7's closed list covers only the bare in-place spellings

**Location:** `_lyx/discussion/decision-record.md` D7.

**Issue:** The scan matches a form only as a whole token sequence, so the entries `sed -i` and `perl -i` do not match `perl -pi -e`, `sed -ni`, `sed -i.bak` or `sed --in-place`, all of which D2 denies.
D7 already treats the `rm` deny as pattern-shaped and outside the scan, and says so, but presents `sed -i` and `perl -i` as if two entries follow D2's deny.
The `PATTERN-no-denied-recovery` constraint bullet then claims no stencil names `sed -i` or `perl -i` as a step, which holds only for the bare spellings.
The gap needs stating, as the raw `go` precedent does, or a plan writer will believe the scan enforces D2.

### [LOW] D3's Bound omits positional parameters

**Location:** `_lyx/discussion/decision-record.md` D3, Bound.

**Issue:** The pattern denies `$` followed by a letter, `_` or `{`.
`rm "$1"`, `rm $@` and `rm $*` are computed paths it does not match, and the lets-through list does not name them.

### [LOW] The Goal claims more prompt-raising forms are steered than D3 covers

**Location:** `_lyx/discussion/decision-record.md` Goal, D3 Rationale and Auto-mode assumptions.

**Issue:** The Goal says "the shell forms that raise one are steered away before they run".
D3 refuses only `rm` with a substitution or variable.
That this matches Claude Code's "statically-unresolvable target" trigger set is assumed, not stated.
The listed assumption covers only that the deny runs before Claude Code's check.
If Claude Code also prompts on another form, the stall still happens, and only D6's notice surfaces it.

### [LOW] D1's exclude entry has no stated reach

**Location:** `_lyx/discussion/decision-record.md` D1, second paragraph.

**Issue:** Excluding `.scratch/` makes untracked files there invisible to `git status`, and so to any dirty-tree check.
That is a bypassable guard under the rubric's attack-surface question.
D1 states neither whether the entry is root-anchored, as `ExcludeAnchoredDir` writes them, or bare, nor what it lets through.

## Rejected

- D5's read-only claim was checked against `splitShellWords` and `readOnlySedSegment` in `internal/fabricengine/refscanner.go` and holds: with `/`, `;`, newline, `$` and backslash excluded from a regex, no admitted address or range (`/a/,/b/p`, `/a/,3p`, `1,5p;20,30p`) can hide a non-print command.
  Redirection, substitution and dash-led file words are already rejected before the sed grammar runs, and every acceptance-criteria example agrees with the stated grammar.
- D3's stated false denies and its `xargs rm` let-through hold against `commandPositionPattern`, which lists `xargs` as a command-position prefix.
- D1 against D3: removal by literal path and the computed-path deny agree, and the directive names no `rm` form.
- Completeness before leanness: every requirement in the Interview and Question ledger (the 60-second bound, the llm-tier live check, the `.git/info/exclude` seeding, `awk` staying a reader) is already in `decision-record.md`; no relocation was found.
- The missing `file:line` and signature detail is not flagged: the writer's stencil forbids gathering it.
