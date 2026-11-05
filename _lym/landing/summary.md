# Report the Codex darn end-to-end trial

Adds `docs/research/codex-engine-trial-darn-e2e-2026-11-05.md`, the report of the sandbox hub's `codex-greet` run, a darn run whose writer is the Codex model `gpt-6-luna`.
The run stopped at Darn-Write: the writer implemented the change, but every `git commit` failed because the worktree's git directory under the main checkout's `.git/worktrees` lies outside the sandbox's writable roots.
The report records the setup, timeline, token cost and each check against the Codex and creel engine docs, and marks what the watch did not verify.
The fault is filed as board note `codex-writer-cannot-commit` in project `codex-engine-trial`.
No code changes.
