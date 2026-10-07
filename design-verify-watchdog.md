# Step-level watchdog and a Windows verify timeout that kills the process tree

`verify-watchdog` · Note · enhancement, shed

From run-recovery PR-Gate (2026-10-07): run-recovery gave `verifytree` a 60-minute timeout and kills the process group on Unix. Still open from #420: a step-level watchdog, so a `lyx shed step` that never returns is noticed without the hub reading panes; and on Windows the timeout kills only the shell, not the process tree it started.