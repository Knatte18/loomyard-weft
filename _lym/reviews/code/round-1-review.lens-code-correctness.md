```yaml
lens: code-correctness
outcome: reported
```

No skill names are validated on the run path either, so an empty `--skill ""` is accepted the same way on both paths.

**Correctness list (what I ran against each function or branch)**

- `buildOpenCmd`, `firstTurn` empty (every pre-existing caller, and the resume line): the line is identical to before. Correct.
- `buildOpenCmd`, `firstTurn` set: the line is `claude '<msg>' --session-id <id> --name …`. The message has no single quote, only backticks and a colon-bearing skill list. Single-quoted POSIX and pwsh quoting keep it literal, and `sh.Quote` handles a skill name containing a quote. The positional-before-flags form is the same one `buildLaunchCmd` already uses for the launch pointer. Correct.
- `OpenLaunch`, `spec.Skills` empty: the message is empty, so the line and behaviour are unchanged. Correct.
- `OpenLaunch`, message over the limit (a skill name of 30000 `x`):
  - The check runs before `creelFlagDirForHook`, `newSessionID` and `writeOpenFiles`, so nothing is created and no directory needs removing.
  - It measures `len(message)` in bytes against `maxLaunchPromptBytes`, the same quantity `Prepare` measures at `claudeengine.go:181`.
  - Quoting adds two bytes beyond the measured length; the run path has the same margin. Correct.
- `OpenLaunch` resume line: `buildOpenCmd(..., "", ..., resume=true)` gives `claude --resume <id> …` with no message, joined by `OrElse` to the fresh line. If resume fails because the session never took a turn, the fresh line runs under the same id and carries the skill-load message. After a skills-opened session has taken its load turn, resume works and does not reload the skills. Matches the doc comments. Correct.
- `Runner.Open`, skills set and the engine has no `SkillLoader`:
  - The check sits after the `SessionOpener` check and before `newRunID`, the marker `MkdirAll`, `OpenLaunch` and `AddStrand`, so a refusal creates nothing.
  - The error wraps the `skillLoader` error and contains `SkillLoader`, which the test row matches.
  - The test row also asserts that the marker directory is absent and `AddStrand` was not called. Correct.
- `Runner.Open`, skills nil: the check is skipped. Correct.
- `shuttlecli open`, `--skill a --skill b`: `StringArrayVar` keeps the order and does not split on commas. Skills go onto the spec after `--model` resolution sets `spec.Engine`, so engine selection happens before `Runner.Open` runs its skills check. `--list-models` returns before any of this. Correct.
- `shuttlecli open`, `--skill ""`: the list becomes `[""]`, and `SkillLoadMessage` yields "…in this order: . Do nothing else…". Open accepts it with no error. The run path has no name validation either, so this matches existing behaviour; it is a boundary gap but not a regression.
- Codex engine, `--skill` set: Codex has no `OpenLaunch` in this tree. Open refuses it with "cannot open a session", before the new skills check. Correct today. The summary's open risk stands: once a Codex `OpenLaunch` lands, nothing forces it to honour `spec.Skills`.
- Reed prime `board` button: `ask_default: board`, and `{answer}` is allowed because an `ask` is present. The run string is `cd {worktree} && lym shuttle open --role {answer} --model 'opus[high]' --skill ly:board-policy --skill ly:board`, the same form as the existing `agent` button plus the two `--skill` flags. The posix template, the test's `wantPrimeButtons` and the same-buttons-in-both-templates subtest agree. I did not read the Windows template diff, only the test's comparison of it. Correct.
- `orchSkills` gains `ly:board-policy`: it is used by `start.go:196` for launch and `watch.go:126` for the watcher's reload, so both paths load it. Correct.

**Findings**

- **LOW, design.** `internal/shuttlecli/open.go` (the `--skill` flag) and `internal/shuttleengine/open.go:74`: an empty skill name is accepted and produces a malformed first-turn load message. Fix: refuse an empty entry in `--skill`, or in `Runner.Open`, before anything is created. I am reporting it only because the open path adds a new entry point for skill names. Evidence: `claudeengine/skillload.go:19-23` joins names with no check.

I found no other correctness defects in this lens.

**Tool calls (1 of 6)**
1. `grep Skills` in `shuttleengine/spec.go` and `run.go`: to see whether the run path validates skill names, which my context lacked.
