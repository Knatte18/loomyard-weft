# Move the orch rules into the hub-wide rules file

`orch-rules-move` · Note · enhancement, orch

This note is an operator step, run from the hub's prime worktree after the orch-rules-files task lands and the deploy has put `lyx orch rules` in production.
It is never promoted to a run: a task pair's call to `lyx orch rules` is refused by the orch's prime-only refusal.

From the prime, draft the hub-wide rules file in `.scratch/` with exactly the text between the two fence lines below, then run `lyx orch rules --from <draft>`.

```markdown
### Production

An exception to `CLAUDE.md`'s Production rule: after your own pushed fix, on a clean `main`, you run `update-plugins.sh` yourself and say so in your report.

### Watching runs

The operator watches each run in its own terminal window, never in VS Code.
VS Code opens only when the operator asks for it, through `lyx ide` or `batten.yaml`'s `open_ide`.
```
