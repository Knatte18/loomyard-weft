```yaml
lens: board-fit
outcome: reported
```

Task: check how the record fits the rest of the hub (board entries, peer runs, parent commits).

**Checked**
- `lym loom peers`: no peer runs and no parent commits, so there is nothing to conflict with on that side.
- Decision notes for this task: `lym board list` has no note titled `Decision for creel-hook-post:`, so I read none and missed none.
- `creel-agents` (F16, note): overlaps the record, findings 1 and 2 below. Its stage-3 scope (stop reports, `lym shuttle send` and `interrupt`, stencils that name `SendMessage`) is no conflict, since the record leaves it out. Its operator decision of 2026-10-27, that it waits until hook delivery is seen mid-turn in a real run, fits this task.
- `codex-engine` (E2, note): overlaps the record, finding 3 below. The record keeps Codex out, as the entry says; no conflict.
- `idle-notice-dead-waiter-cause` (B1, note): no overlap. It is about the waiter process behind shuttle's activity reader, not creel's mail or a hook waiter.
- `pane-input-lost-on-respawn` (B7): went by its title and brief only. No overlap, since it concerns typed operator input on a re-spawn, not creel posts.

**Findings**

1. MEDIUM · `consistency` · D6 (reminders and notices go) against `creel-agents`' "Reply discipline" section.
   - The chase design in `creel-agents` rests on the reminder path that D6 removes. Its orch recommendation says "The chase follows the reminder rules: typed only at idle, no message content".
   - Its "Orch view" also rests on the reminder: "the reminder that starts its turn is itself typed into the pane like an operator line".
   - After this task lands, that note describes a mechanism that no longer exists, and nothing in the record says so.
   - The writer can settle this. Add a line under Open risks or Notes saying that `creel-agents`' chase design assumed the reminder path D6 removes, so a chase there would need its own typed line, like D5's bell. Or file a board note pointing this out. The note's body itself is the operator's to change.

2. LOW · `consistency` · D9 and D10 (reply rule in the system prompt and help) against `creel-agents`' reply-discipline idea.
   - `creel-agents` proposes that `lym creel read` print the reply command with each message, plus a shared stencil block stating the rule.
   - D9 and D10 already deliver the core of that, in the system prompt and the Cobra help, and say nothing about the overlap.
   - Nothing contradicts. The fix is a line saying that D9 and D10 meet part of `creel-agents`' reply discipline, and that the expected-reply flag and chases stay there.

3. LOW · `consistency` · D5 and D6 against `codex-engine` step 1.
   - D6 removes the pane path for posts for every strand, not only Claude strands.
   - A future Codex strand therefore gets mail only once `codexengine` renders a tool-boundary hook and an idle wake. With D5, it would also get a bell it cannot act on.
   - The record says Codex is out of scope and the seam stays neutral, but it does not say that `codexengine` must now bring creel delivery hooks to receive mail at all.
   - The fix is one line under Constraints or Notes: a new engine receives creel mail only through its own hooks, so `codex-engine` step 1 inherits that need.

No `decision` findings: no landed commit or other run contradicts the record, and the entry has not been overtaken by landed work. The latest related commits, `3422ceac`, `5ad172e7` and `546bc89c`, are the ones the entry builds on.

**Tool calls**
- `lym loom peers`: the peer runs and parent commits, which my context did not hold.
- `lym board get creel-agents`: the body of the nearest touching note, since my context held only its brief.
- `lym board get codex-engine`: the body of the Codex note, to check that keeping Codex out fits it; my context held only its brief.
- `lym board get idle-notice-dead-waiter-cause`: its body, to rule out an overlap with D4's waiter; my context held only its title.
