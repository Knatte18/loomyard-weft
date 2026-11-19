# Raddle checks pass a part slug that Raddle-File then refuses as too long

`raddle-slug-length-check` · Task · bug, shed · starting · Seed-IDE

## Seen

2026-11-19: raddle's proposal for `single-repo-records-branches` passed its checks and the operator approved it, but Raddle-File refused the part `single-repo-records-on-code-removal` because its slug is 35 characters and the board's limit is 32.
The run stopped with a question to the parent.

## Idea

The proposal checks apply the board's slug rules, the length limit included, so a slug too long is caught before the operator approves.
