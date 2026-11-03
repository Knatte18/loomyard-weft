# drift capture child rejects its cwd when the fabric anchor is a repo subdirectory

`drift-child-error-in-halt` · Note · enhancement, drift

Imported from [issue #26](https://github.com/Knatte18/loomyard/issues/26).

## What happened

`lym loom` halted as `failed` at the `Drift-Baseline` producer (anomaly `producer-hard-failure`) four times in one task, on each entry and each `lym loom resume`.
Every halt carried the message "drift: the capture child ended before it recorded its start".
The capture child logs under `_lym/drift/logs/` each hold the same single line, an error of the form:

`cwd is outside the recorded fabric anchor subtree: cwd <worktree root> does not equal the anchored directory <worktree root>/src/csharp/NORCE.Models (recorded by .lym-anchor)`

The fabric anchor (`.lym-anchor`) records a subdirectory of the worktree, but the capture child was started with the worktree root as its cwd, so its anchor check rejected it.
The driver's resume advice, "read the log, then `lym loom resume`, whose entry captures again", cannot succeed because the cwd does not change between attempts.
The run only got past the producer once it was skipped (`Drift-Baseline / skipped` in the history).

## Why it is abnormal

The anchor is lym's own record, and the capture child is lym's own process.
They disagree about the working directory, so a task whose anchor is not the worktree root can never capture a drift baseline.
The halt message also points at a log whose one line is the real cause, so the operator learns it only after opening the log.

## What to do

- Start the capture child with its cwd set to the anchored directory, or have the anchor check accept the worktree root when the anchor lies beneath it.
- Surface the child's error text in the halt message instead of only naming the log.
- Fail the producer once with a non-retry hint when the child's cwd error repeats, rather than advising a resume that cannot change the outcome.

Provenance: task `testsupport-rule-and-helpers`; notes `loom-halt.md`, `loom-halt-2.md`, `loom-halt-3.md`, `loom-halt-4.md`.
