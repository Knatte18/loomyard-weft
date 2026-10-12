---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (pattern extent and Bound), D7, Acceptance criteria (in-place deny row)
    summary: GNU sed accepts any unambiguous abbreviation of --in-place (--i, --in, --in-pl=.bak), which D2 neither denies nor names among the forms its Bound lets through
    fix: In D2, widen the long form from `--in-place` and `--in-place=<suffix>` to any `--` word that is a prefix of `--in-place` at least as long as `--i`, optionally followed by `=<suffix>`, and say in one clause that no other GNU sed long option starts with `--i`, so the widening denies no read. In the in-place deny acceptance criterion, add `sed --in s/a/b/ f` and `sed --in-pl=.bak s/a/b/ f` to the refused list. In D7, add the abbreviated long forms to the spellings it says stay outside the token scan.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D6 (persisted offset and attach rebuild paragraph), Acceptance criteria (reattach row)
    summary: D6 persists only the asks folded before the notice is sent, so an ask of the same dialog that lands after the send is rebuilt into a latch on reattach and repeats the notice, which the reattach criterion says never happens
    fix: In D6, after the sentence that persists the latest ask's offset before the notifier is called, add that while the latch stays set after its notice, each further ask folded into it advances the same persisted dialog offset, so a reattach never rebuilds a latch from an ask of an episode already notified. Extend the reattach acceptance criterion with a case whose `permission_prompt` `Notification` lands after the notice was sent and before the restart, and which sends no second notice.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D6 (latch bullets), Auto-mode assumptions (60-second bullet)
    summary: The latch reads the pane only once it is 60 s old, so a dialog answered at once keeps the latch set, and a second dialog raised just before the bound is notified seconds after it appears, contradicting the assumption that an operator answering a dialog at once sees no notice
    fix: In D6's third latch bullet, read the pane through the input-box reader on each poll tick from the moment the latch is set, not only once it is `permissionNoticeBound` old; a read showing an input box clears the latch at any age, so a later ask opens a new episode stamped with its own time and offset, and the notice is still sent only by a no-input-box read once the latch is bound-old. Add an acceptance criterion where an ask is followed by an input box at 10 s and a second ask at 55 s, and no notice is sent before 60 s after the second ask.
---

### [MEDIUM] D2 misses abbreviated `--in-place` spellings

**Location:** `_lyx/discussion/decision-record.md` D2, D7 and the in-place deny acceptance criterion.

**Issue:** D2 lists the in-place spellings it denies as `-i`, `-i<suffix>`, `--in-place`, `--in-place=<suffix>` and short clusters holding `i`.
GNU sed parses long options through `getopt_long`, which accepts any unambiguous prefix.
Run in the session scratchpad with GNU sed 4.9, `sed --in s/a/b/ f` rewrote `f` in place, and `sed --in-pl=.bak s/b/c/ f` rewrote it and left `f.bak`.
Neither spelling holds `--in-place` or a single-dash cluster, so the deny lets both through, and D2's Bound lists neither among what it lets through, though the rubric's attack-surface rule requires the Bound to state what the guard lets through.
`--in-place` is the only GNU sed long option starting with `--i`, so denying every prefix of it from `--i` up admits every read form, `--quiet` and `--silent` included.

The rest of focus 4 holds.
A flag word opening after whitespace with a single dash keeps `--quiet`, `--silent`, `--expression=` and `--file=` out of the short-cluster match.
The `-i<suffix>` and `-ni`/`-Ei` cases are denied, and the word extent ends at a separator or `\n`, so `grep -i` after a newline is outside it.
D7 says honestly that only the bare `sed -i` and `perl -i` reach the token scan.

### [LOW] D6 reattach can repeat a sent notice for a late second ask event

**Location:** `_lyx/discussion/decision-record.md` D6, the persisted-offset and attach paragraph, and the reattach acceptance criterion.

**Issue:** Focus 3, traced: dialog A raises `PermissionRequest` at offset o1, the latch is set, and at 60 s the notice is sent with o1 persisted.
A's `permission_prompt` `Notification` then lands at o2 > o1.
The live loop folds it into the set latch, so nothing is sent, but the persisted offset stays o1.
The driver restarts while A is still up, and the attach rebuild takes the asks after o1, finds o2, and builds a latch that is already bound-old.
The first pane read shows no input box, so A's notice is sent a second time.
D6's claim that persisting the latest ask "covers them all" holds only when every ask event of a dialog lands before the send.
No assumption states that ordering, and the reattach criterion promises no repeat for a `PermissionRequest` followed by a `permission_prompt` `Notification` with no timing condition.
A distinct dialog folded into a notified episode, such as a second fork's dialog while the first is still up, takes the same path.

Focus 2, traced, holds: once the notice is sent, D6 keeps reading the pane on each poll tick until an input box shows, which clears the latch, and a second ask then sets a new latch with its own stamp and offset, whose notice the second-notice acceptance criterion pins.

### [LOW] D6's latch outlives a quickly answered dialog until the bound

**Location:** `_lyx/discussion/decision-record.md` D6's latch bullets and the 60-second Auto-mode assumption.

**Issue:** The latch clears only through a pane read, and the third bullet reads the pane only once the latch is `permissionNoticeBound` old.
Take a dialog answered at 10 s, and a second dialog raised at 55 s.
The second ask lands while the latch is still set, so it changes nothing, and the 60 s read sees the second dialog and sends the notice 5 s after it appeared.
The Auto-mode assumption says 60 seconds is long enough that an operator answering a dialog at once sees no notice; this sequence breaks it.
Reading the pane from the moment the latch is set costs one input-box read per poll tick, and only while a dialog may be up.

## Focus verdicts

- Focus 1, D1 against `deniedRecoveryMarker`: not a defect.
  `deniedRecoveryTokenPattern` splits on backticks, commas and whitespace.
  In "never `sed -i`, never `perl -i`, never `awk`, and never `python` or `node` in an inline heredoc", the text before each `sed`, `perl` and `python` token ends in "never " plus one backtick, the shape `forbiddenByMarker` exempts.
  "Reading with `sed -n`" matches no closed-list entry, and `editRulePhrases`' new "`sed -i`" is a substring of the directive's spelling, so the single-statement check still finds it.
  A grep of the scanned surface (`contracts/`, `plugins/ly/skills/` and the non-test Go under `internal/webster*`, `internal/shed*` and `internal/loom*`) finds no other `sed -i` or `perl -i` today.
- Focus 2: holds; traced under F2.
- Focus 3: yes, the notice can repeat; see F2.
- Focus 4: one in-place spelling class is missing; see F1.
