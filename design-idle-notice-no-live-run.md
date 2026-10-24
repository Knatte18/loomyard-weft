# Batten idle notice says no agent activity readable while a run is live

`idle-notice-no-live-run` · Task · bug, batten, shuttle · running · Finalize

## Seen (2026-10-24)

The task `idle-notice-misses-fork-activity` quoted Batten's idle notice as "no agent activity readable" for a writer working in a fork.
Its fix (c5d715860) closed the fork gap in `ReadAgentActivity`, which yields "agents idle", not that text; Batten writes "no agent activity readable" only when it finds no live run at all.
If the quote was exact, a second cause hides a live run from the activity reader.

## Idea

Find which path made the reader see no live run while the writer worked, and fix it; check the run records of a run that hit the notice before closing this as a misquote.