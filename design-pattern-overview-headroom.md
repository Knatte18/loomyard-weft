# PATTERN overview breaks its cap only after merging main

`pattern-overview-headroom` · Note · enhancement, loom

From `gate-test-scope` (2026-10-10): the `PATTERN.md` overview was under its 16384-byte cap on both the task branch and main, but Publish merged main in and the overview reached 16559 bytes.
`TestOverview_RealFilePassesCheck` then failed in verify, the run blocked at Publish, and it needed a parent decision plus a goto to Webster-Burler.
The same block then hit `daemon-wakeups` and `bolt-and-store` the same day, and `daemon-wakeups` landed at 16337 bytes although a parent decision asked for at most 16200: a target only in prose does not stick.

Idea: give the overview headroom, with the overview check warning at about 95% of the cap, or let a shared size cap breached only by the merge be fixed through the Publish fix route.
