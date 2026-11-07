# codexengine: read a skill loaded when a tool output holds its SKILL.md

A Codex seat that read its skills through a path it built at run time was logged `skill skipped ... cause="not loaded"`.
The classifier only looked for the `skills/<name>/SKILL.md` tail in the tool calls' payloads.
A seat that runs a loop such as `` `cat .../skills/${name}/SKILL.md` `` in one `exec` call never writes the tail out, so the call payload does not name it.
The warning was false, and it hid a true one.

`ClassifySkillLoad` in `internal/shuttleengine/codexengine/skillload.go` now also reads a skill as loaded when a tool call output of the load turn holds the whole of its SKILL.md.
Both `function_call_output` and `custom_tool_call_output` count, whether the output is a string or a list of text blocks.
The name-in-a-call test is unchanged and still decides first.

## Design decisions

- The file is read from the path `skillPath` resolves under codex.yaml's `skill_roots`, and its content is matched with surrounding whitespace trimmed.
  An empty or unreadable file is never read as printed.
- A partial output does not load a skill, so a model that printed only the head of the file stays reported missing.
- Only calls and outputs after the latest load message count, as before.
  A repeated load message restarts both.
- `loadTurnCalls` becomes `readLoadTurn`, which returns a `loadTurn` holding the calls and the outputs.
  The text-block decoding that `messageText` used is shared with the output decoding.
- `doc.go` and the `skillload.go` comments describe the second way a skill reads as loaded.

## Tests

`TestClassifySkillLoad` gains cases for a read through a built path (string output and split text-block output) and for a partial output.
The existing before-the-message and repeated-message cases now also carry an output, so they pin that an earlier output does not load a skill.

## To check by hand

- The original report's rollout, `~/.codex/sessions/2026/11/06/rollout-2026-11-06T13-29-27-01a1b0d5-*.jsonl`, was not used.
  Its `exec` input was not shown to hold a built path, so the cause fits that rollout but is not confirmed against it.
  Run the classifier on it and check that the five skills read as loaded.
- No integration or live Codex run was done.
