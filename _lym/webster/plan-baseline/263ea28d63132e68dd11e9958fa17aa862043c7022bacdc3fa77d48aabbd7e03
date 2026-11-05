# Card 1 — pattern-clusters

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-child-processes.md`

**Extends:**
- `pattern/PATTERN-agent-addressing.md`

**Intent:** Fold the task's two PATTERN rules into the live clusters, as the parent's two Added-after-Discussion decisions record, before any card leans on them.

In `PATTERN.md`, replace the `PATTERN-child-processes` and `PATTERN-agent-addressing` overview lines with these lines, verbatim:

```markdown
- `PATTERN-child-processes` — Starting or ending a process: log the spawn via `internal/logger`; its starter waits on it, bar a double fork's intermediate, which exits at once; a run's processes register in `internal/runproc` and end only through it. — [background](pattern/PATTERN-child-processes.md)
- `PATTERN-agent-addressing` — Naming or messaging an agent: names only via `internal/agentname`; messages only through creel; Slack only via `internal/slackengine`, its token only in its machine credential file; operator Slack text reaches agents only as creel mail from `<shortname>:SLACK`. (test) — [background](pattern/PATTERN-agent-addressing.md)
```

No top-level `PATTERN-reap-children` or `PATTERN-slack-boundary` entry is added.

In `pattern/PATTERN-child-processes.md`, the opening summary sentence takes the same exception, and the `## Reap children` rule becomes: "Starting a child process: its starter waits on it, by `cmd.Wait` or `proc.Reap`, never only releasing it, bar a double fork's intermediate, which exits at once; a process that re-execs itself calls `proc.ReapInheritedChildren` before it starts a child."
Below the rule, one paragraph says what the exception lets through and its bound: the intermediate of a double fork starts the real process in a new session and exits without waiting on it, so the process is reparented out of the starter's tree (the Slack relay, D3); the starter still waits on the intermediate, and the released process is reaped by whatever it is reparented to, which rests on no lym process being a child subreaper.

In `pattern/PATTERN-agent-addressing.md`, add a `## Slack boundary` section: Rule: Calling Slack: only `internal/slackengine` calls the Web API, reading the bot token only from its machine credential file (`slack.json` in githubclient's credential directory), never from a config layer, `lym config env`, an environment variable or a repository; operator Slack text reaches an agent only as creel mail from `<shortname>:SLACK`, and an agent reaches Slack only by creel mail to that program.
It is held by review alone; no test enforces it.

**ImpactSummary:** Two overview lines change text in place and two background files gain the exception and the Slack section; no code and no test reads the changed wording.
