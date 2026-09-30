The hub orchestrator's context fills over a long day; compaction loses detail, and the manual remedy is `/scribe:handoff` then a fresh session.
Proposal: `lyx hub orch` (name open) runs the orchestrator as an interactive claude strand in its own reed session, and a watcher cycles it:
1. Read context usage from the session transcript (usage fields) after each turn.
2. Above a threshold, when the session is idle (shuttle's idle/asking classification), inject `/scribe:handoff <path>` and wait for the file.
3. Inject `/clear`, then a resume prompt that reads the handoff and the board.
Durable state already lives outside the context (board queue and dependencies, run status files, drive reports, `manifest/HANDOFF.md`), so a fresh session loses little.
The operator talks to the orchestrator through `lyx reed attach` or the hub VS Code workspace terminal.
Open: threshold, whether the operator's own terminal session can be adopted instead of spawning a new one, and cross-session messaging to a strand-hosted orchestrator.