# The orch ends its own earlier Claude process and resumes its session, never launching fresh over a live holder

`orch-resume-own-session` · Task · bug, batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle · running · Worktree-Create

Decision record: [Sketch](decision-orch-resume-own-session.md)

Filed from the Sketch `adopt-redesign`.

Takes: internal/orchcli/start.go's launchOrch with its fresh-launch fallback, runproc's roles and End functions, claudeengine's CheckResume, the orch watcher's owned-elsewhere read and shuttleengine's carry-over; PATTERN-test-only-code holds, so every new function has a production caller
Delivers: the orch's Claude process registered in runproc under an orch role keyed by the worktree; an orch start that ends its own registered holder and resumes the same session id, refuses an unregistered holder naming its pid, and launches fresh with the carry-over only when no local transcript exists, saying so; the orch watcher ending every process of a subject owned elsewhere, the dobby last
Exports: runproc's orch role and its registration call, and one function that ends every process this machine runs for a subject, which the orch watcher calls here and the dobby's handover and the catch-up call in bolt-holds-catch-up

The part changes these packages:

- internal/orchcli (about 350 changed lines)
- internal/runproc (about 120 changed lines)
- internal/orchengine (about 50 changed lines)
- internal/shuttleengine (about 30 changed lines)
