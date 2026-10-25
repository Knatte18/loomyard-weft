## Goal

Find out, on its own and before creel becomes the one router, whether a tool-boundary hook can deliver a strand's mail in full, for Claude Code and for Codex.
The output is a report with measured results and a recommendation for note `creel-carry-agent-mail`.
Nothing goes into production.

## Scope

- A prototype under `.scratch/`: a flag file per strand, a mail file per strand, and one hook script that tests the flag and, when it stands, hands over the mail.
- A stand-in for creel's side (post, flag set, flag clear, idle-path delivery) built in the same scratch directory; no change to `internal/creelengine` or `internal/creelcli`.
- Test strands: real Claude Code and Codex sessions in a private tmux server, in a throwaway repository under `.scratch/`.
- Claude: a `PostToolUse` hook against one long turn (several tool calls, one long) and against an idle agent reached by pane paste.
- Codex: whether `PostToolUse` fires, else whether `PreToolUse` can carry the mail.
- Measurements: arrival and exactly-once delivery when both paths race, delay against Claude Code's SendMessage, hook cost per tool call with and without mail, and how the receiver treats the mail.
- The deliverable is `docs/research/creel-hook-delivery-2026-10-25/report.md`.
- No hook is installed in the operator's or the plugins' settings; hooks are given per session by settings file or flag under `.scratch/`.

## Decisions

### Prototype lives in `.scratch/`, report in docs

The prototype hook and test strands stay under `.scratch/` (never tracked); only the report is committed.
Rationale: the board entry asks for exactly this, and a prototype that is not production code carries no PATTERN or doc.go obligations.

### Creel's side is simulated with files, not the daemon

The flag and mail files are written by a small scratch script that plays creel's post and delivery rules.
Rationale: the question is whether the hook path works and what it costs, which does not depend on the daemon; a change to creel waits on the recommendation.
Bound: the simulation proves nothing about daemon races, so exactly-once is measured for the file protocol (claim by atomic rename) and the report names that limit.

### Exactly-once by atomic claim

The hook and the idle-path deliverer both claim a message by renaming its file into a `claimed/` directory; the loser of the rename delivers nothing.
Rationale: the mechanism a real creel would need is a single-owner claim, and rename is atomic on one filesystem.

### Real agents, cheapest models

Claude sessions run on Haiku and Codex sessions on `gpt-6-luna` at low effort, each prompt a few lines.
Rationale: the measurements are about delivery, not model quality; the receiver-treatment check may use one run on a stronger model if the cheap one is ambiguous.

### Hook gives mail as hook context, framed as a strand message

The hook output wraps each message as `[creel] #<id> from <sender>: <body>` in the hook's additional-context channel, with no reminder line.
Rationale: this matches the notice format the receiver already knows and carries the id for `lyx creel post --reply`.
Bound: it lets agent text reach another agent's context without the reminder guard; the report weighs that against the saved turn, and nothing here ships.

## Constraints

- `CLAUDE.md`: work only in this worktree, never push, never install a hook in user or plugin settings, never run `update-plugins.sh`.
- The edit directive: files change by Edit or Write only; scratch files go under `.scratch/`.
- Agents run in interactive tmux, never `claude -p`; Codex likewise runs interactive.
- Markdown: one sentence per line.
- No Go production code changes, so no PATTERN entry applies beyond the above.

## Auto-mode assumptions

- A darn run holds no interview; the board entry is the whole scope.
- Claude Code 2.1.310 and Codex CLI 0.164.0 on this machine are the versions tested; the report names them.
- The operator's Claude and Codex logins work for private-server sessions.

## Open risks

- Codex may fire neither `PostToolUse` nor a hook whose output the model sees; then the report says Codex cannot take mail mid-turn and the idle path stays its only route.
- A hook-trust or folder-trust prompt in Codex may need an Enter; the harness presses it, as the spike found.
- Delay figures depend on model latency and tool durations; the report states the conditions of each run and does not claim more than the runs show.
- Quota use by real sessions; runs are kept short.

## Acceptance criteria

- The report exists at `docs/research/creel-hook-delivery-2026-10-25/report.md` and is committed.
- Claude mid-turn: every message posted during one long turn reaches the receiver at a tool boundary, without an interrupt, and the report states the delay.
- Claude idle: a message posted to an idle strand arrives by the pane path.
- Exactly-once: with both paths racing for the same message, the receiver sees it once.
- Cost: the no-mail hook's time per call and the with-mail time are measured and stated.
- Codex: the report states whether `PostToolUse` fires and whether the model sees a hook's output from `PreToolUse` or `PostToolUse`.
- Receiver treatment: the report states whether the receiver took the mail as a strand message and not an operator instruction, and whether it answered by `lyx creel post --reply`, or what it did instead.
- The report ends with a recommendation for `creel-carry-agent-mail`.
- No file outside `docs/research/creel-hook-delivery-2026-10-25/` and `.scratch/` changes.

## Notes for the plan writer

None; this is a darn run.
