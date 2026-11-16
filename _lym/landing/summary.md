# Report web research on using Claude Code's Code Mode well

Adds `docs/research/claude/code-mode-web-research-2026-11-16/report.md`,
which follows the Code Mode bench on the Plan writer.
The bench found no gain.
This report gathers what Anthropic and users say Code Mode is for and how to get gains from it,
then judges each tip against lym's writers and names the bench that would test it.

## Sources

- Anthropic: the Code Mode docs page, the 2.1.323 and 2.1.325 changelog entries,
  the 2025 posts on code execution with MCP and on programmatic tool calling, and the API docs for programmatic tool calling.
- The `REPL` and `Poll` tool prompts shipped in the installed Claude Code 2.1.325 binary, read with `strings`.
- The bench's own Code Mode writer transcripts, read with `jq`.
- Hands-on reports: wmedia.es, two third-party plugins that imitate Code Mode, and the release coverage.
- Reddit, fetched through a Codex session from the Arctic Shift archive, since reddit.com refused every request with HTTP 403.

Every source is listed with its URL and date.
Two Medium posts refused the fetch and are cited by title only.

## Findings

- Nothing found would turn the Plan writer result into a gain.
  Anthropic's guidance names reasoning over each intermediate result, which is the Plan writer's work,
  as the case where programmatic tool calls do not help.
- The bench report blamed the flat turn count on each `REPL` call needing a `Poll` call.
  That is wrong: the writers sent `Poll` in the same turn as `REPL` in 36 of 46 and 15 of 21 turns.
  The turns stayed level because each turn ends at a judgment, and the control already batched.
  The bench report's verdict bullet now points to the correction.
- lym launches sessions with `--dangerously-skip-permissions`, so its writers get the form where `REPL` dispatches and `Poll` collects.
  In a subagent, `REPL` waits and returns its output.
- Users report gains on long, read-heavy work, from tool schemas and intermediate results kept out of the context.
  They report writes as no better or worse, since one script error sinks a batch of edits.
- Later benches worth running:
  - a darn task with repeated mechanical edits, control against Code Mode;
  - a Plan-Write arm told to filter inside its scripts;
  - before any adoption, a check that lym's shell guard refuses a guarded command run through a script's `$` template.

`docs/research/README.md` gains a `claude` section that indexes this report and the bench report, which had no entry.
No production code changed and no bench ran.
