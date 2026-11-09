# Console window titles put the task handle first, with hub and project in brackets

`console-title-handle-first` · Note · enhancement, termwindow

## Seen (operator, 2026-11-09)

Since `7bccde26` every console window is titled `<hub>: <P#>: <T#>: <slug>`, such as `ly: P1: T2: adopt-worker-record`.
The operator finds it hard to read: the task handle, which they look for first, sits third, behind two colon-separated prefixes.

## Idea

Put the task handle first and set the hub and project apart in brackets, for instance:

`T2 [ly:P1] adopt-worker-record`

How the hub and project are shown is still open; the operator only fixed that the handle comes first and that brackets read better than a chain of colons.
Decide the form for the windows that have no task (a hub's prime, a beam, the runs window) and for a task under no project, in the same style.
