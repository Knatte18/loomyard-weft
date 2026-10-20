# Edit directive not enforced: file append through a shell heredoc was not refused

`shell-guard-heredoc-write` · Note · bug, claude

Imported from [issue #772](https://github.com/Knatte18/loomyard/issues/772).

**What happened**
An agent appended a section of a markdown doc through a shell heredoc instead of Edit or Write, against the prompt's edit directive.
The shell guard did not refuse it; the content was later corrected with Edit.

**Why it is abnormal**
The directive forbids scripted file rewrites, and other commands in the same session were refused by the guard, so enforcement is uneven.

**What to do**
Extend the shell guard to refuse heredoc and redirect writes (`cat >> file <<EOF`) to tracked files under the edit directive, or confirm and document the intended gap.

Provenance: task `gate-fewer-new-binaries`, note `Darn-Write.md`.
