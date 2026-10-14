# Support log — run-window-bugs

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick, recorded in the question ledger.

- Turn 1 — read the board entry and the board.
  The entry bundles six notes plus a teardown race; no other entry duplicates them.
  `reed-multi-window` has landed (`58534d414`), so shuttle run strands default to `own-window`.
- Turn 2 — explored the startup probe (`awaitStartup`, `checkLivenessTick`, `Startup`), the launch line (`launchPointer`, skills defer the pointer to `PromptLine`), the status pane loop, the push retry, the socket tests, the edit directive and the teardown sequence.
- Turn 3 — settled the readiness design: events proof in `start`'s probe, paired with the launch-line change so every `start` launch has a turn start.
- Turn 4 — settled the remaining items: full clear in `statuspane`, the weft finding, the shared stale-socket helper, `lyx loom fmt`, the bounded removal retry.
- Turn 5 — lens review by six forks; every fork returned a report.
  - pattern-fit, BLOCKING: the launch-line skill load changes `PATTERN-role-skills-typed` unnamed — fixed: D2 names it, rewords it, and lists `docs/overview.md` and the test comment.
  - pattern-fit, MEDIUM: `lyx loom fmt` is hub-only while the directive renders everywhere — fixed: the line is worded for Go files, and D6 notes a repo without Go files never needs it; agents run in hubs, where loom resolves.
  - pattern-fit, LOW: where D7's retry lives — fixed: inside `fabricengine`'s contained removal, with an injected sleep.
  - pattern-fit, LOW: a temp file for the in-place write — fixed: D6 writes in place, so no temp file arises.
  - pattern-fit, LOW: share one formatting function between the verb and `gofmt_test.go` — rejected: both call `go/format.Source`, which already is the one function.
  - pattern-fit, LOW: docs list lacked `docs/overview.md` and `PATTERN.md` — fixed.
  - premises + goal-scope + scenarios, MEDIUM (merged): D3 (now D2) misstated the color's order — fixed: wait for the load turn's end, then color, retry turn, pointer; acceptance covers a long load turn.
  - premises + goal-scope, MEDIUM (merged): a re-entered removal does not finish a directory holding only recreated locks — fixed: D7's retry repeats the contained removal inside the one call, after its checks passed.
  - premises + goal-scope + scenarios, MEDIUM (merged): D4 (now D3) asserted the root cause and left the resize-to-poll window — fixed: the cause moved to assumptions and risks, and a resize wakes the loop at once.
  - premises, LOW: "too short" applies only to an engine that reads a box — resolved by dropping the old D2.
  - premises, LOW: name signal kinds, not hook names — fixed in D1.
  - scenarios, MEDIUM: the old D2's resumed-launch capture path is not reached by `start`, since a resume spec carries the pointer — fixed: old D2 dropped; Wait's and attach's probe is out of scope.
  - scenarios, MEDIUM: stale events on relaunch or attach could prove readiness — fixed: D1 applies only in `start`'s probe, with a fresh events file.
  - scenarios, MEDIUM: `lyx loom fmt` symlinks, mixed batch, no args, file mode — fixed in D6 and acceptance.
  - scenarios, LOW: not-empty matched by text — fixed: matched by error value.
  - scenarios, LOW: an attach loop would defeat the bounded retry — added to Open risks.
  - scenarios, LOW: negative scenarios for D1 and the launch-carried load turn — added to acceptance.
  - scenarios, LOW: clearing history drops copy-mode position — added to Open risks.
  - goal-scope, MEDIUM: the weft finding rests on reasoning, and Warn-to-Info hides the signal — fixed: the log level stays Warn, the finding goes to the doc comment, and the three Publish sightings' round cost moved to assumptions.
  - goal-scope, LOW: held sends in too-short panes not explicitly out — fixed in Scope.
  - goal-scope, LOW: the edit directive's location — fixed: `contracts/stencils/edit/edit-directive.md`.
  - board-fit, LOW: `go-driver-assistant` edits the same shuttle files — fixed: named in Notes.
  - board-fit, LOW: Warn-to-Info against `spawn-audit-log`'s log levels — moot: the level no longer changes.
  - board-fit, LOW: window-0 risk naming the driver — moot: that risk left with the narrowed scope.
  - board-fit, LOW: the teardown choice replaces the entry's two named options — rejected: the reasons are in Rejected alternatives, and the record carries no rejected alternatives.
  - leanness, MEDIUM: D8 (now D7) argued against the marker alternative — fixed: the clause is cut; the re-entry sentence that stays is a premise the plan needs.
  - leanness, LOW: D5 (now D4) diagnosis narrative — rejected in part: the diagnosis is the deliverable the entry asks for; the rejected fetch-and-rebase clause is cut.
  - leanness, LOW: Goal restates Scope — fixed: Goal is one sentence.
  - leanness, LOW: Scope Out reasons — fixed: boundaries only.
  - leanness, LOW: test-budget note restates `CLAUDE.md` — fixed: cut.
  - leanness, LOW: generic docs rule — fixed: only the task's list stays.
  - leanness, LOW: D2 board-provenance sentence — resolved by dropping the old D2.

## Rejected alternatives

- Minimum pane height in reed's layout: own-window placement already gives run strands full windows, and the events proof removes `start`'s dependence on the capture.
- Capturing scrollback in the startup probe: a resumed session redraws its transcript, whose past prompts could read as a ready marker.
- An unreadable capture deferring to the session-start record: after the launch-line change no `start` launch reaches the capture path without a first turn, so the rule pinned a path nothing takes.
- Events proof in Wait's and attach's probe: their events file may hold an earlier incarnation's lines.
- Readiness from `SessionStart` alone: a session start can precede the TUI accepting input.
- Fetch and rebase before the first weft push: the refusal is a remote race, not a stale ref, and the retry absorbs it.
- A hub lock serializing weft pushes: couples board writes and run publishes for a refusal the retry already handles.
- Demoting the retry log to Info: it is the signal that surfaced the race, and a log-level setting planned in `spawn-audit-log` could hide Info lines.
- Allowing `gofmt -w` in the edit directive: needs a `gofmt` binary on every OS and opens a broader rewriting tool.
- `lyx gate fmt`: `gate` runs tests in hub slots; formatting needs no slot.
- Teardown kills every reed process bound to the pair: no portable way to enumerate them.
- A teardown marker reed checks before creating its lock: the marker lives in the tree being removed, so a writer after its deletion still recreates the lock.
- Retrying the whole `Remove` from `pairteardown`: a re-entered `Remove` treats the leftover directory as a stray path and leaves it.

## Review rounds

- Round 1 — burler review, BLOCKING, six findings, all fixed.
  - F1, BLOCKING: D7's retry sat on the fallback the sighted failure never reaches, since `git worktree remove` deletes the registration before its delete fails — fixed: D7 places the retry in `removeWarpWorktreeDir`'s not-a-linked-worktree branch, with a new ownership proof, a two-part not-empty recognition and per-OS error values; the re-entry sentence and the acceptance criterion are reworded.
  - F2, MEDIUM: D1's tool-event proof named a signal no hook records — fixed: the proof is a session start followed by a turn start.
  - F3, LOW: D1 cited D3 for the launch-line change — fixed: D2.
  - F4, LOW: D2 did not bound the bypass of the verified send — fixed: a Bound line names the skipped landing check, the turn-end proof and the skill-load timeout measured from the end of `awaitStartup`.
  - F5, LOW: D6 left root resolution, a partial write and three bounds untested — fixed in D6 and acceptance.
  - F6, LOW: Scope item 4 had no acceptance criterion — fixed.
- Round 2 — burler review, APPROVED, three findings, all fixed.
  - F1, MEDIUM: D7's Windows value missed a lock file still held by its writer, which fails the removal with a sharing violation — fixed: `windows.ERROR_SHARING_VIOLATION` is retried beside `windows.ERROR_DIR_NOT_EMPTY`, with a Windows acceptance criterion.
  - F2, LOW: D7's recognition named a git delete failure only stderr could decide — fixed: the first half is the branch condition, nonzero exit, unregistered and still on disk.
  - F3, LOW: D7's leftover removal recorded no mutation — fixed: it appends `KindPathRemoved` with detail `recursive`, Constraints names `PATTERN-mutation-record`, and the acceptance criterion checks the entry.

## Question ledger

- Q1 — Does readiness still need the capture when events prove a turn started? Auto-pick: no, events first in `start`'s probe (D1).
- Q2 — How should a collapsed strand not read as never-ready? Auto-pick: the events proof on every `start` launch (D1 with D2); the earlier capture-deferral rule was dropped after review.
- Q3 — Include the launch-line first-turn change despite its low priority? Auto-pick: yes; it is on the bundled entry and gives D1 a turn start on every launch (D2).
- Q4 — Weft push: fix or log? Auto-pick: the retry is the intended path; record the finding, change nothing else (D4).
- Q5 — Where does the shared stale-socket helper live? Auto-pick: a test kit, chosen by the plan (D5).
- Q6 — Verb or directive exception for gofmt? Auto-pick: verb `lyx loom fmt` plus one directive line (D6).
- Q7 — Teardown race fix shape? Auto-pick: bounded retry inside the contained removal (D7).
