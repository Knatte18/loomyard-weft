# tokencount: guard the slug-reuse cutoff against date skew

`tokencount-cutoff-skew` · Note · enhancement

Seen at the PR-Gate review of small-fixes-oct11 (PR #556, 2026-10-11): `tools/tokencount` now takes a cutoff from the slug's archive tags and skips main-session transcripts that start before it, with no flag to turn it off.
A tag whose commit date is skewed against the transcript timestamps could drop sessions silently; the report only says how many it skipped.

Idea: watch for this in the tokencount-full-runs measurements, and if it shows, add a flag to disable the cutoff or a tolerance window.