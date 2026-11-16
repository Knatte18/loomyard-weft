```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the reviewer's proposals the fixer carried out: all of F2 and F3, plus the parts of F1 the fix kept (the Facts rewrite and the rule that a `function_call` ends a chain).

Proposals checked:
- **F2** (a launch-spec literal is one that sets `Prompt`; inline prompts get their render split out): I found no case where it goes wrong. The tackler and arbiter attach probes are built by spec builders whose literal sets `Prompt` from a parameter, so they are launch functions and correctly stay in the table. No production launch literal leaves `Prompt` out and assigns it afterwards: the only `.Prompt =` assignment is on a non-Spec type in `claudeengine/capabilities.go:72`.
- **F1** (the parts kept as proposed): I found no case to build. No `wait` or other `functions`-namespace tool appears as a `function_call` in the rollouts, so the chain-ending rule has no counterexample on record.
- **F3** (resolve both paths to absolute; the earlier read counts only when not older than the file's last change): one finding, below.

Findings:

1. **MEDIUM.** Changed text: D12's read-again bullet, "the sliced read's against the payload's `cwd`".
   - Summary: a nested `exec_command` can run in its own `workdir`, so the payload's `cwd` is not the directory a relative path in the command resolves from.
   - Evidence: the `exec` tool schema in every rollout declares `workdir?: string`. The 2026-11-15 lym rollouts show the model setting it, as `workdir:"/home/knatte/Code/loomyard-LYMHUB/codex-writer-bench-neve…"` and `workdir:cwd`. The spike payload carries `tool_input` with `command` only, and its `cwd` is the session directory. The spike never set a `workdir`, so nothing on record shows the payload `cwd` following one.
   - Concrete case: the model runs `tools.exec_command({cmd:"sed -n '1,5p' notes.txt", workdir:"/repo/sub"})` in a session whose cwd is `/repo`. The rule resolves the path to `/repo/notes.txt`, the wrong file. The result is a nudge on the wrong file, or none at all.
   - Only the earlier read's side is safe, because its `CommandExecution` item carries the call's own `cwd`. The sliced read's side needs the same source, which may be unwritten when `PostToolUse` runs, or the record has to state the bound.

Tool calls beyond context:
- One grep for `workdir` across the four Codex rollouts. My context lacked whether `exec_command` takes a per-call working directory and whether the model uses it.
