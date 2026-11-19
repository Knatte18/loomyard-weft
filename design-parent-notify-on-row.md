# Mail the run parent when the run reaches a named row

`parent-notify-on-row` · Note · feature, loom, shed

## What was seen

From a comment on issue #153 (2026-11-19): a parent that wants to brief the agent of a given row as it starts has to poll `lym loom status --watch`.

## Idea

Let the parent subscribe to a run's state, so lym mails it when the run reaches a named producer or state:
for example `lym loom notify <slug> --on Drift-Verify,Drift-Expect,Code-Burler-Fix`, or a `run_settings` key such as `notify_parent_on: [Drift-Verify]`.
The parent can then brief each agent of those rows as it starts, such as telling the drift judge to put its questions to the parent.
