---
round: 1
focus:
  - "internal/shuttleengine/editloop.go `fold` and `watchEditLoops`: do the per-agent, per-file count, the sliding window and the count restart after a notice match the board entry's 'past a set count in a set window'? Check edit_loop_edits of 1, out-of-order or zero signal times, and a map that grows with every file edited."
  - "internal/shuttleengine/editloop.go `watchEditLoops` and wait.go: does the persisted EditLoopNotifiedOffset keep a reattach from re-notifying or dropping edits, and what happens to edits folded into a batch whose offset write failed? Check the cursor start against PromptOffset, and the pause and interactive guards."
  - "internal/shuttleengine/claudeengine/editrecord.go `RecordEdit` and the hook command in settings.go: is the events-file append safe against concurrent hooks and the wait loop's partial-line reads? Does the stamp-then-record pair parse back to one SessionSignalFileEdit with the right time and AgentID, and is the hook added only for an autonomous run with the watch on?"
  - "internal/shuttlecli/hookedit.go and cli.go: does the hidden `hook-edit` verb fail soft, run outside a hub, resolve the engine by registry name, and leave the session free of hook errors? Check the argument handling and exit codes."
  - "internal/shuttleengine/config.go, template.yaml and ConfigSpec: do the defaults, the validation of edit_loop_edits and edit_loop_window_s, and the config category agree with each other, and does the docs update the repository's rules require land in the same diff (the doc.go files, any spec the constraints name)?"
  - "Scope against the board entry: the entry asks for counting edits per file per agent session from PostToolUse events and a friction note to the parent without killing the session. Does the diff add anything beyond that, such as the new engine interface, the new signal fields or the notice wording, that the entry does not license, or omit any part of it?"
  - "Commit message and `_lym/landing/summary.md` claims against the diff: the first commit's description says the hook is slim and the session keeps running; check each claim."
  - "Test economy: the new tests are all listed as unique in the redundancy report, so check instead for behavior the diff adds with no coverage (the persist-failure path, reattach, the interactive guard, the subagent notice wording) and for assertions on static prose."
---

Seed focus from the diff stat and a read of `editloop.go`, `editrecord.go` and the settings, config, engine and wait hunks.
The decision record holds only the board entry under `## Goal`, so the entry is the measuring stick.
