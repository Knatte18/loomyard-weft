# creel: a mailbox between named lyx processes

`creel` · Someday · feature

The mail store and verbs from `manifest/designs/reed-mailbox.md` (send, inbox, read, log; the watchdog delivering at idle; `inject` for privileged senders), addressed by `agent-naming`'s names.

Deferred (operator, 2026-10-02): Claude Code's own SendMessage is enough for now.
The address grammar in reed-mailbox.md (`name@slug[.hub]`) is superseded by `agent-naming`'s `<repo>:<slug>:<role>`, and its naming half moved into that task.
The design doc goes with `manifest/` in `bolt-v2`; this note keeps what is left of it.

Use case (2026-10-03): the operator's backup session on Remote Control rescues the orch, for example by re-enabling Remote Control after a cycle, and today has to find reed's per-hub tmux socket by hand; `inject` to a named strand covers it.
