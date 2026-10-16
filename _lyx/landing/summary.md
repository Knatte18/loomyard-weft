# Fix creel stage 1 audit defects and add sandbox scenarios C1-C13

Creel now reaches hand-added Claude strands, says why a strand cannot take a delivery, loses no mail on a failed read, and prints consistent output.
The audit's sandbox scenarios C1 to C13 join the shuttle sandbox suite beside S9, so the audit can be re-run.

Closes #657
Closes #658
Closes #659
Closes #660
Closes #661
Closes #662
Closes #663
Closes #664

## What changes

- **#657, placeholder read as a draft.**
  Reed gains a styled pane capture beside `CapturePane`, and shuttle gains an optional engine capability that turns it into a plain capture with the provider's placeholder cells blanked.
  `claudeengine` implements it: a box whose text is drawn wholly in Claude's placeholder style reads as empty.
  This applies to every Claude run's send path (idle probe, pre-write guard, landing check, creel's interrupt reads), not only hand-added strands.
- **#658, silent stall.**
  The Delivery seam's idle answer carries a stall reason (pane too short, or box holds a draft).
  The engine journals one `stalled` record per change of reason per receiver, and `log` prints it.
  `inject --wait` gets a progress line on stderr per change, and its envelope carries the last reason and its time; an `expired` record carries the reason too.
- **#659, restored prompt after interrupt.**
  Shuttle gains an interrupt variant for creel that clears the prompt Claude restores into an empty box.
  Both `lyx creel interrupt` and `inject --interrupt` use it; `Runner.Interrupt` and its other callers are unchanged.
  The receiver's delivery worker is suspended while it runs.
- **#660, lost mail.**
  The read verb checks its write and returns the error, so nothing is acknowledged and the messages stay unread.
- **#661, lost daemon error.**
  The daemon's stderr goes to a per-hub log declared in `hubgeom`'s creel Geometry.
  A give-up error names the file and quotes the lines from the spawned daemon's start.
- **#662, TTL.**
  Each inject is charged TTL from its own posting, not from the worker's previous probe.
- **#663, #664.**
  `read` prints `ended_unread_by` and `ended_unread_at`.
  `log` prints `reminded` ids and step.
  Help texts state the privilege rule as `doc.go` does.
  Waiting verbs print the resolved sender.
  `release` ends the caller's hold first and refuses an unknown name, listing the names that exist.

Docs updated in the same change: the `doc.go` of `creelengine`, `creelcli`, `shuttleengine` and `reedengine`, the `claudeengine` comments, `PATTERN-pane-text-injection` and `PATTERN-guarded-interrupt`.

## Design decisions to review

- A box is blanked only when every non-blank cell, after stripping the caret, side bars and a lone `%`, carries the placeholder style.
  Typed text, typed text beside placeholder-styled text, and the collapsed-paste shape are never blanked.
  A Claude release that restyles the placeholder falls back to today's fail-closed reading (not idle).
- The restored-prompt clear is its own sequence in `claudeengine`: one `C-u` per line of the read text with one `BSpace` between each pair, and no Escape, `C-c` or Enter.
  `ClearInputSequence` is unchanged.
  It runs at most once per interrupt, behind the same three-fact guard as the stray-character clear, only when the box was empty before the key and holds text after.
  A failure after the key is journaled as `interrupted` with `sent:true`, and the `inject --interrupt` line is still queued.
- The log file is truncated past a fixed cap.
  A concurrent start's truncation can erase another start's lines; the give-up error then says the file holds no line from that start.
  Daemon logger lines carry the pid so interleaved starts stay apart.
- The `stalled` record and the `expired` field are ignored by an older binary's fold.

## To check by hand

- The restored-prompt clear sequence and the placeholder style are pinned from captured fixtures (`testdata/pane-restored-prompt-styled.txt`, `testdata/pane-placeholder-styled.txt`); confirm against a live Claude session that the caret sits at the restored prompt's end and a multi-line prompt is emptied.
- `TestSmokeCreelClaudeStrand` (`lyx gate test --tags llm ./internal/creelcli`) was reported failing in one run on Claude Code's folder-trust dialog, judged environmental and not verified; run it with the folder trusted.
- `sgr-faint-reads-extended-colour-subparameters` in `claudeengine/styledcapture.go`: check that the SGR parse does not read extended-colour subparameters as the faint attribute.
- `daemon-log-unreadable-branch-unreached` in the daemon log open path: check that the branch for an unopenable log file (daemon starts with null stderr, give-up error says so) is reached and covered.
- The sandbox scenarios C1 to C13 were written but not run; run them against a live hub.
- C12's disagreeing-strand-name check is operator-assisted when the agent's harness refuses a command naming the strand variable.
