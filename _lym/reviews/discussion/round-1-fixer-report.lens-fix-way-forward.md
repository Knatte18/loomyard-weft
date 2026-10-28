```yaml
lens: fix-way-forward
outcome: reported
```

The fix's ways forward checked:
1. D1: `post` refuses a line past 8 KiB, with the file-and-path way forward.
2. D1: an old-binary message past 10,000 characters stays for `lym creel read` and shows in `lym creel inbox`.
3. D2: at the cap the `Stop` claims nothing, then D4's ring delivers.
4. D2: a restarted daemon starts every count at the cap, and a reset or ring recovers.
5. D4: the ring reads `NoTurn`.
6. D5: the bell is held by a draft, a too-short pane or a missing input box, and is limited to strands marked `CreelHooks`.
7. D7: a receiver without the mark is refused, with two ways forward named.
8. D7: a store that fails after the key gets an error.
9. D9: `append-system-prompt.txt` sits beside the run's `settings.json` and is passed on the launch and resume lines.
10. Open risks: pre-deploy sessions get delivery once restarted.
11. The new acceptance criteria.

Findings:

- MEDIUM. D7, the new refusal sentence: "naming the ways forward: post without `--interrupt` and have the receiver run `lym creel read`, or stop its turn with `lym creel interrupt`."
  Neither way forward gets the message to the receiver.
  Interrupting an unmarked strand and then posting delivers nothing; that is the case the refusal exists for, and D6's new Bound says such a strand gets no signal of any kind.
  "Have the receiver run `lym creel read`" names no channel the sender could use to make that happen.
  The working way forward is `lym creel inject --interrupt <receiver> <text>`, which stops the turn and types the text.
  D6 keeps injects, and the sender rules match: `internal/creelcli/cli.go`'s inject help says "EXTERNAL callers, programs and orch strands may inject, interrupt and hold on any strand, and any other strand on its own strand only", which is the same rule `interrupt` follows.

- MEDIUM. D9, the new text: "Shuttle writes the appended text … to `append-system-prompt.txt` beside the run's `settings.json`."
  D9 still says "Every Claude session shuttle launches", and the auto-mode assumption still says "standalone ones included".
  A `lym shuttle open` session has neither of those files.
  `internal/shuttleengine/open.go`'s `Open` doc says "It writes no run directory, no run record and no output file".
  `buildOpenCmd` says it "passes no prompt, no --settings (the session takes the worktree's own settings)".
  So the step cannot be carried out for open sessions, and D9 names no location for their file.

- LOW. D7, the new sentence: "A store that fails after the key was sent answers an error saying the turn was interrupted and the post was not stored."
  The error names no way forward, such as re-running `lym creel post` without `--interrupt`.
  Other creel refusals in this record each name one.

- LOW. Open risks, the edited line: "they get no `Stop` delivery, no waiter, no bell and no D9 sentence until restarted."
  A restart through reed's verbatim resume replays the old resume line with the old run's `settings.json`.
  Only `Prepare` renders settings (`internal/shuttleengine/claudeengine/claudeengine.go:208-213`), and D5's new text says the mark is "kept through reed's verbatim resume".
  So only a fresh shuttle launch gives such a session the new delivery, and "restarted" does not say so.

Tool calls:
- Read `internal/creelcli/cli.go`'s inject and interrupt help, because my context lacked their sender rules and `inject`'s `--interrupt` behaviour.
- Grepped where `buildSettings` and `Prepare` write `settings.json`, because my context lacked whether a resume re-renders settings.
