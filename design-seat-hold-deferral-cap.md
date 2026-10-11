# Seat chair hold deferral of 20 min can hide a wedged advisor

`seat-hold-deferral-cap` · Note · enhancement, shuttle

Seen at burler-multi-llm PR-Gate (PR #552): `holdNoticeMaxDeferral` defers a seat chair's held notice up to 20 min while advisor work is outstanding, so an advisor stuck mid-turn hides a real chair stall for up to 20 min.

Idea: shorten the cap once runs show typical advisor batch times, or end the deferral when the advisor itself is held or dead.