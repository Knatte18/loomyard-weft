# Console window titles put the task handle first, with hub and project in brackets

`console-title-handle-first` · Note · enhancement, termwindow

## Seen (operator, 2026-11-09)

Since `7bccde26` every console window is titled `<hub>: <P#>: <T#>: <slug>`, such as `ly: P1: T2: adopt-worker-record`.
The operator finds it hard to read: the task handle, which they look for first, sits third, behind two colon-separated prefixes.
A task under root shows no project today.

## Idea

Put the leading handle first and set the hub and project apart in brackets:

- A task: `T2 [ly:P1] adopt-worker-record`.
- A task under root names root's project, `P0`: `T5 [ly:P0] machine-name-file`.
- An orch leads with `ORCH` in place of a task handle:
  the prime's orch is `ORCH [ly:P0]`, and a beam's orch is `ORCH [ly:P1] adopt-takeover`, naming its beam's project and slug.

Fixed by the operator: the handle comes first, brackets read better than a chain of colons, and root is project `P0`.
Still open: the exact form of the bracket, and the title of the other windows that hold no task, such as the runs window, in the same style.
