# Console window titles put the task handle first, with hub and project in brackets

`console-title-handle-first` · Task · enhancement, termwindow

## Seen (operator, 2026-11-09)

Since `7bccde26` every console window is titled `<hub>: <P#>: <T#>: <slug>`, such as `ly: P1: T2: adopt-worker-record`.
The operator finds it hard to read: the task handle, which they look for first, sits third, behind two colon-separated prefixes.
A task under root shows no project today.

## Design (operator, 2026-11-09 and 2026-11-11)

Put the leading handle first and set the hub and project apart in brackets, as `<lead> [<hub>:<project>] <name>`:

- A task: `T2 [ly:P1] adopt-worker-record`.
- A task under root names root's project, `P0`: `T5 [ly:P0] machine-name-file`.
- An orch leads with `ORCH` in place of a task handle:
  the prime's orch is `ORCH [ly:P0] Prime`, and a beam's orch is `ORCH [ly:P1] adopt-takeover`, naming its beam's project and slug.

There is no runs window to title; every console window is a task's or an orch's.
