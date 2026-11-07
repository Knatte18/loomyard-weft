# Codex skill load: count a skill read through a built path as loaded

## Cause

The Codex seat in the `assist-fan-seat-work-alone` Code-Review did load its skills.
In both load turns of rollout `rollout-2026-11-06T13-29-27-01a1b0d5-*.jsonl`, the model made one `exec` `custom_tool_call` whose script loops over the skill names and runs ``cat …/skills/${name}/SKILL.md``.
The matching `custom_tool_call_output` holds every SKILL.md in full.
`ClassifySkillLoad` counted a skill loaded only when a call's payload held the literal `skills/<name>/SKILL.md` tail.
The template literal never holds that tail, so all five skills read as missing after the retry and were logged `cause="not loaded"`.
The payload kind and the turn boundary were both read correctly.

## Change

`internal/shuttleengine/codexengine/skillload.go` now also collects the tool call outputs after the latest load message (`function_call_output` and `custom_tool_call_output`, as an output string or joined text blocks).
A skill counts as loaded when a call names its tail, or when one output holds the whole of its resolved SKILL.md, with surrounding whitespace trimmed.
A partial read through a built path still counts as missing, and an output before the load message does not count.
The package doc states the second way a skill reads loaded.

## Tests

`TestClassifySkillLoad` gains a `ReadThroughABuiltPath` row modelled on the real rollout:
a script call with `${name}` in its path, a block output holding one file whole and a string output holding part of another.
The `ReadBeforeTheMessage` row now also has an output before the load message.
The fixture skill files hold more than a heading, so a partial output differs from the whole file.
