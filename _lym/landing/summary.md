# Study: the work to support Gemini CLI or Antigravity CLI as an engine

Adds `docs/research/engines/gemini-antigravity-engine-study-2026-11-17.md`, a desk study, and its entry in `docs/research/README.md`.
No code changes.

## What the report holds

- **Verdict.**
  Neither tool fits lym yet, and Google's terms are the first blocker.
  - Gemini CLI is the cheaper build: it is open source, its hooks are shaped like Claude Code's,
    and it takes a per-run settings file and a session id chosen up front.
    But since 2026-06-18 it serves only Code Assist Standard / Enterprise and paid API or Vertex keys,
    and it is not tuned for Gemini 4.
  - Antigravity CLI (`agy`) is where Google's models now go, and its Stop hook has `continue` and `fullyIdle`.
    But it is closed source, and its FAQ says wrapping `agy` inside third-party agents is not supported under consumer plans.
  - If a Google engine is wanted, the target is Antigravity CLI on an API key or Enterprise account,
    once the operator has settled the terms and a spike has been run.
- **The engine seam, mapped from the code.**
  - The core interface and the optional capabilities.
  - The fork and background-wake requirements, and the roles that need them.
  - Every file outside the engine package that a third engine touches.
    `internal/shuttleparity` names exactly two engines, so it needs reworking first.
- **An item-by-item table** for Codex today, Gemini CLI v0.68.0 (2026-11-10) and Antigravity CLI v1.5.10 (2026-11-17).
  It judges each item native, workaround or missing.
  The items are pane launch and readings, trust, model and effort, per-run config, events, deny guards,
  creel delivery and idle wake, edit records, batching nudge, skills, instructions and heddle targets,
  forks, background wake, resume, usage, liveness, session commands, sandbox and headless mode.
- **The Codex yardstick.** For each landed Codex task: its commit, and its lines of non-test Go, test Go and other files.
  The tasks are split into seam work, build and parity.
  No record of the tokens or time spent building them exists, and the report says so.
- **An estimate per candidate**, in lines and landed tasks, scaled from Codex's per-engine share.
  - Gemini CLI: 5.5 to 6.5k lines of non-test Go and 12 to 15 tasks.
  - Antigravity CLI: 7 to 8.5k lines and 15 to 20 tasks.
- **A first minimal level**, the single-session, fork-free roles Codex reached first, and what it leaves out.
- **A list of what documents could not settle.**

## Design decisions

- The report starts a new `engines` topic in the research index, since it covers neither Claude nor Codex alone.
- The terms texts, Gemini CLI's status notice and Antigravity's hook schema were read from the fetched pages themselves.
  The rest of the per-tool findings come from reading the documents and source, not from a running tool.
  The report says which is which.
