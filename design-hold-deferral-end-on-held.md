# End the seat hold deferral when the advisor itself is held or dead

`hold-deferral-end-on-held` · Note · enhancement, shuttle

Seen in shuttle-startup-ready (PR #557, 2026-10-11): Part 3 shortened `holdNoticeMaxDeferral` from 20 to 10 minutes (`hold.go`).
The alternative from the same idea was not built: end the deferral as soon as the advisor the chair waits on is itself held or dead, instead of waiting out the cap.