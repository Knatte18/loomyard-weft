# creel: a hand-added claude strand is never read idle, so nothing is delivered (llm smoke test fails)

`creel-hand-added-claude-idle` · Note · bug, creel, shuttle, reed, claude

Imported from [issue #657](https://github.com/Knatte18/loomyard/issues/657).

Found by the creel stage 1 audit (board note `creel-test`), in a sandbox hub, with `lyx` built from `main` at e131c7625.

## What was seen

A strand added by hand with `lyx reed add --cmd claude` is never read idle, so creel never types a reminder or an inject into it.
A fresh Claude Code session shows its greyed placeholder in the empty input box, for example `❯ Try "edit <filepath> to..."`.
A pane capture carries no styling, so `claudeengine.IdleSession` reads that placeholder as a draft and answers not idle.
Shuttle-launched sessions avoid this because every shuttle settings file sets `promptSuggestionEnabled: false` (`internal/shuttleengine/claudeengine/settings.go`); a hand-added strand gets no such settings.

The llm-tier smoke test fails the same way: `lyx gate test --tags llm ./internal/creelcli` runs `TestSmokeCreelClaudeStrand`, which fails twice in a row with `smoke_llm_test.go:51: the adapter never read the booted session idle`.
The test adds the strand with the bare claude path, as the commit's "hand-added strand is reachable" claim intends.

## Steps

1. In a sandbox hub: `lyx reed add --role alice --cmd claude` (here `claude --model haiku`), accept the folder-trust prompt.
2. `lyx creel post ctest:alice "please say READ"`.
3. Watch the pane for minutes: no `[creel]` line is ever typed; `lyx creel log` shows `posted` and nothing else.
4. Control: the same strand launched as `claude --settings '{"promptSuggestionEnabled":false}'` shows an empty box and gets the reminder within a second.

## Expected

A strand added by hand with `--cmd claude` is reachable by creel, as `internal/creelengine/doc.go` and the commit message say, and `TestSmokeCreelClaudeStrand` passes.
Either creel/shuttle reads a placeholder-only box as empty, or `lyx reed add` (or the docs) makes hand-added Claude strands start with prompt suggestions off.
