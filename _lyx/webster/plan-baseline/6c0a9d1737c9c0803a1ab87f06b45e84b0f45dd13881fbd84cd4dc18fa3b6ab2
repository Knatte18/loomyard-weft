# Card 4 — orch-rules-move-note

**Create:**
- `.scratch/orch-rules-move.md`

**Intent:** File D4's board note, slug `orch-rules-move`, so the operator's step to move today's two rules into the hub-wide file is on the board when this task lands.
No card can move them: the only route that commits the hub-wide file is `lyx orch rules`, which reaches production only with the deploy after landing and runs only from the prime.

Write the body to `.scratch/orch-rules-move.md`, run `lyx board upsert '{"slug":"orch-rules-move","kind":"note","title":"Move the orch rules into the hub-wide rules file","brief":"Operator step after the orch-rules-files deploy: run lyx orch rules from the prime","labels":["enhancement","orch"]}' --body-file .scratch/orch-rules-move.md`, then remove `.scratch/orch-rules-move.md` by its literal path.
The note writes another board entry, never this task's own, and the card commits no file: its one target is the git-ignored draft the upsert reads, created and removed within the card.

The body, verbatim between the two `~~~` lines, holds the step and the rules text D4 fixes, the old H1 dropped and its H2s written as `###`:

~~~markdown
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
~~~
